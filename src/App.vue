<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from "vue";
import { invoke } from "@tauri-apps/api/core";
import { getIdentifier, getName, getTauriVersion, getVersion } from "@tauri-apps/api/app";
import { emit, listen, type UnlistenFn } from "@tauri-apps/api/event";
import { appDataDir, homeDir, join, tempDir } from "@tauri-apps/api/path";
import { getCurrentWindow } from "@tauri-apps/api/window";
import { openUrl } from "@tauri-apps/plugin-opener";
import {
  AppWindow,
  BookOpen,
  CheckCircle2,
  CircleDashed,
  FolderSearch,
  Hammer,
  MonitorCog,
  RefreshCcw,
  Rocket,
  Signal,
  XCircle,
} from "lucide-vue-next";

type TestState = "idle" | "running" | "pass" | "fail";

interface TestResult {
  state: TestState;
  summary: string;
  durationMs?: number;
}

interface TestCase {
  id: string;
  title: string;
  category: string;
  run: () => Promise<string>;
}

const reveal = ref(false);
const launching = ref(false);
const logLines = ref<string[]>([]);
const testResults = ref<Record<string, TestResult>>({});

onMounted(() => {
  requestAnimationFrame(() => {
    reveal.value = true;
  });
});

const unlistenList: UnlistenFn[] = [];

onBeforeUnmount(() => {
  unlistenList.forEach((off) => off());
});

const tests: TestCase[] = [
  {
    id: "invoke-greet",
    title: "Invoke Rust Command",
    category: "core",
    run: async () => {
      const reply = await invoke<string>("greet", { name: "Tauri API Probe" });
      if (!reply.includes("Tauri API Probe")) {
        throw new Error("unexpected greet reply");
      }
      return reply;
    },
  },
  {
    id: "app-meta",
    title: "Read App Metadata",
    category: "app",
    run: async () => {
      const [name, version, identifier, tauriVersion] = await Promise.all([
        getName(),
        getVersion(),
        getIdentifier(),
        getTauriVersion(),
      ]);
      return `${name} ${version} (${identifier}) / tauri ${tauriVersion}`;
    },
  },
  {
    id: "path-basic",
    title: "Read Runtime Paths",
    category: "path",
    run: async () => {
      const [appDir, home, temp] = await Promise.all([appDataDir(), homeDir(), tempDir()]);
      const reportPath = await join(appDir, "reports", "api-check.json");
      return `home=${home} | temp=${temp} | report=${reportPath}`;
    },
  },
  {
    id: "event-loop",
    title: "Emit And Listen Event",
    category: "event",
    run: async () => {
      const eventName = `api-check-${Date.now()}-${Math.random().toString(36).slice(2)}`;
      const payload = `ok-${Date.now()}`;
      let off: UnlistenFn | undefined;

      const received = new Promise<string>((resolve, reject) => {
        const timeoutId = setTimeout(() => {
          reject(new Error("event timeout"));
        }, 1500);

        void listen<string>(eventName, (event) => {
          clearTimeout(timeoutId);
          resolve(event.payload);
        })
          .then((unlisten) => {
            off = unlisten;
            unlistenList.push(unlisten);
            return emit(eventName, payload);
          })
          .catch((error: unknown) => {
            clearTimeout(timeoutId);
            reject(error);
          });
      });

      const actual = await received;
      off?.();
      if (off) {
        const index = unlistenList.indexOf(off);
        if (index >= 0) {
          unlistenList.splice(index, 1);
        }
      }

      if (actual !== payload) {
        throw new Error(`payload mismatch: ${actual}`);
      }
      return `event payload received: ${actual}`;
    },
  },
  {
    id: "window-info",
    title: "Read Window Info",
    category: "window",
    run: async () => {
      const appWindow = getCurrentWindow();
      const [scale, size, fullscreen] = await Promise.all([
        appWindow.scaleFactor(),
        appWindow.innerSize(),
        appWindow.isFullscreen(),
      ]);
      return `label=${appWindow.label} | ${size.width}x${size.height} @${scale} | fullscreen=${fullscreen}`;
    },
  },
];

for (const item of tests) {
  testResults.value[item.id] = {
    state: "idle",
    summary: "Not started",
  };
}

const passCount = computed(() => Object.values(testResults.value).filter((it) => it.state === "pass").length);
const failCount = computed(() => Object.values(testResults.value).filter((it) => it.state === "fail").length);

function pushLog(message: string): void {
  const ts = new Date().toLocaleTimeString();
  logLines.value = [`${ts}  ${message}`, ...logLines.value].slice(0, 30);
}

async function runSingle(test: TestCase): Promise<void> {
  const startedAt = performance.now();
  testResults.value[test.id] = { state: "running", summary: "Running..." };

  try {
    const summary = await test.run();
    const durationMs = Math.round(performance.now() - startedAt);
    testResults.value[test.id] = { state: "pass", summary, durationMs };
    pushLog(`${test.title} passed in ${durationMs}ms`);
  } catch (error) {
    const durationMs = Math.round(performance.now() - startedAt);
    const summary = error instanceof Error ? error.message : String(error);
    testResults.value[test.id] = { state: "fail", summary, durationMs };
    pushLog(`${test.title} failed: ${summary}`);
  }
}

async function runAll(): Promise<void> {
  launching.value = true;
  pushLog("Start full API diagnostic");
  for (const test of tests) {
    await runSingle(test);
  }
  pushLog("Full API diagnostic finished");
  launching.value = false;
}

async function openDocs(): Promise<void> {
  await openUrl("https://v2.tauri.app/reference/javascript/api/");
}

function resultBadge(state: TestState): string {
  if (state === "pass") return "text-emerald-700 bg-emerald-50 ring-emerald-200";
  if (state === "fail") return "text-rose-700 bg-rose-50 ring-rose-200";
  if (state === "running") return "text-amber-700 bg-amber-50 ring-amber-200";
  return "text-slate-600 bg-slate-100 ring-slate-200";
}

function stateLabel(state: TestState): string {
  if (state === "pass") return "PASS";
  if (state === "fail") return "FAIL";
  if (state === "running") return "RUNNING";
  return "IDLE";
}
</script>

<template>
  <main
    class="relative min-h-screen overflow-hidden bg-[radial-gradient(circle_at_top_right,_#fef3c7_0%,_#fffbeb_30%,_#eef2ff_65%,_#e0f2fe_100%)] px-4 py-6 sm:px-8 sm:py-10"
  >
    <div class="pointer-events-none absolute -left-10 top-8 h-44 w-44 rounded-full bg-cyan-200/40 blur-3xl"></div>
    <div class="pointer-events-none absolute -right-16 top-20 h-56 w-56 rounded-full bg-amber-200/50 blur-3xl"></div>

    <section
      class="relative mx-auto flex w-full max-w-6xl flex-col gap-4 rounded-3xl border border-slate-200/70 bg-white/85 p-4 shadow-[0_30px_100px_-40px_rgba(15,23,42,0.35)] backdrop-blur md:p-6"
      :class="reveal ? 'translate-y-0 opacity-100' : 'translate-y-3 opacity-0'"
    >
      <header class="flex flex-col gap-3 sm:flex-row sm:items-center sm:justify-between">
        <div class="space-y-2">
          <div class="inline-flex items-center gap-2 rounded-full bg-slate-900 px-3 py-1 text-xs font-semibold tracking-wide text-white">
            <Rocket class="h-3.5 w-3.5" />
            Tauri OHOS Frontend Probe
          </div>
          <h1
            class="text-2xl font-bold leading-tight text-slate-900 sm:text-3xl [font-family:'Space_Grotesk','Noto_Sans_SC',sans-serif]"
          >
            API Test Board
          </h1>
          <p class="text-sm text-slate-600">
            A compact dashboard to verify runtime connectivity for major Tauri JavaScript APIs.
          </p>
        </div>

        <div class="flex flex-wrap items-center gap-2">
          <button
            class="inline-flex items-center gap-2 rounded-xl border border-slate-300 bg-white px-3 py-2 text-sm font-medium text-slate-700 transition hover:border-slate-400 hover:bg-slate-50 disabled:opacity-60"
            type="button"
            :disabled="launching"
            @click="openDocs"
          >
            <BookOpen class="h-4 w-4" />
            API Docs
          </button>
          <button
            class="inline-flex items-center gap-2 rounded-xl bg-slate-900 px-3 py-2 text-sm font-medium text-white transition hover:bg-slate-700 disabled:cursor-not-allowed disabled:bg-slate-500"
            type="button"
            :disabled="launching"
            @click="runAll"
          >
            <RefreshCcw class="h-4 w-4" :class="launching ? 'animate-spin' : ''" />
            {{ launching ? "Running" : "Run All" }}
          </button>
        </div>
      </header>

      <section class="grid grid-cols-2 gap-3 md:grid-cols-4">
        <article class="rounded-2xl border border-slate-200 bg-slate-50 p-3">
          <p class="text-xs uppercase tracking-wide text-slate-500">Total</p>
          <p class="mt-1 text-2xl font-bold text-slate-900">{{ tests.length }}</p>
        </article>
        <article class="rounded-2xl border border-emerald-200 bg-emerald-50 p-3">
          <p class="text-xs uppercase tracking-wide text-emerald-700">Pass</p>
          <p class="mt-1 text-2xl font-bold text-emerald-700">{{ passCount }}</p>
        </article>
        <article class="rounded-2xl border border-rose-200 bg-rose-50 p-3">
          <p class="text-xs uppercase tracking-wide text-rose-700">Fail</p>
          <p class="mt-1 text-2xl font-bold text-rose-700">{{ failCount }}</p>
        </article>
        <article class="rounded-2xl border border-sky-200 bg-sky-50 p-3">
          <p class="text-xs uppercase tracking-wide text-sky-700">Window</p>
          <p class="mt-1 flex items-center gap-2 text-sm font-semibold text-sky-700">
            <AppWindow class="h-4 w-4" />
            main
          </p>
        </article>
      </section>

      <section class="grid gap-3 lg:grid-cols-[1.4fr_1fr]">
        <div class="space-y-3">
          <article
            v-for="test in tests"
            :key="test.id"
            class="rounded-2xl border border-slate-200 bg-white p-4 transition hover:-translate-y-0.5 hover:shadow-md"
          >
            <div class="flex items-start justify-between gap-3">
              <div>
                <p class="text-xs uppercase tracking-wide text-slate-500">{{ test.category }}</p>
                <h2 class="mt-1 text-base font-semibold text-slate-900">{{ test.title }}</h2>
              </div>
              <span
                class="inline-flex items-center gap-1 rounded-full px-2 py-1 text-xs font-semibold ring-1"
                :class="resultBadge(testResults[test.id]?.state ?? 'idle')"
              >
                <CircleDashed v-if="testResults[test.id]?.state === 'running'" class="h-3.5 w-3.5 animate-spin" />
                <CheckCircle2 v-else-if="testResults[test.id]?.state === 'pass'" class="h-3.5 w-3.5" />
                <XCircle v-else-if="testResults[test.id]?.state === 'fail'" class="h-3.5 w-3.5" />
                <Signal v-else class="h-3.5 w-3.5" />
                {{ stateLabel(testResults[test.id]?.state ?? 'idle') }}
              </span>
            </div>

            <p class="mt-2 min-h-10 break-all text-sm text-slate-600">
              {{ testResults[test.id]?.summary }}
            </p>

            <div class="mt-3 flex items-center justify-between">
              <p class="text-xs text-slate-500">
                {{ testResults[test.id]?.durationMs ? `${testResults[test.id]?.durationMs} ms` : "-" }}
              </p>
              <button
                type="button"
                class="inline-flex items-center gap-1 rounded-lg border border-slate-300 px-2.5 py-1.5 text-xs font-medium text-slate-700 transition hover:border-slate-400 hover:bg-slate-50 disabled:opacity-60"
                :disabled="launching"
                @click="runSingle(test)"
              >
                <Hammer class="h-3.5 w-3.5" />
                Run
              </button>
            </div>
          </article>
        </div>

        <aside class="space-y-3">
          <article class="rounded-2xl border border-slate-200 bg-white p-4">
            <h3 class="flex items-center gap-2 text-sm font-semibold text-slate-800">
              <MonitorCog class="h-4 w-4" />
              Runtime Notes
            </h3>
            <ul class="mt-3 space-y-2 text-sm text-slate-600">
              <li class="flex items-start gap-2">
                <FolderSearch class="mt-0.5 h-4 w-4 text-sky-600" />
                Path APIs validate environment and app data locations.
              </li>
              <li class="flex items-start gap-2">
                <Signal class="mt-0.5 h-4 w-4 text-amber-600" />
                Event APIs verify message bus reachability.
              </li>
              <li class="flex items-start gap-2">
                <AppWindow class="mt-0.5 h-4 w-4 text-emerald-600" />
                Window APIs validate current runtime window channel.
              </li>
            </ul>
          </article>

          <article class="rounded-2xl border border-slate-200 bg-slate-900 p-4 text-slate-100">
            <h3 class="text-sm font-semibold">Execution Log</h3>
            <div class="mt-3 max-h-64 space-y-2 overflow-auto pr-1">
              <p v-if="logLines.length === 0" class="text-xs text-slate-300">No logs yet.</p>
              <p v-for="line in logLines" :key="line" class="text-xs leading-5 text-slate-300">
                {{ line }}
              </p>
            </div>
          </article>
        </aside>
      </section>
    </section>
  </main>
</template>