# شرح الـ Verification Plan النهائي — CV32E40P (RV32IM)

> **الملف ده لمين؟** لأي مهندس جديد في الفريق لسه بيتعلم Verification من الصفر.
> **المكتوب بإيه؟** شرح تدريبي كامل من الأول للآخر، بالمصري، والمصطلحات التقنية بالإنجليزي زي ما هي في البلان.
> **المصدر الوحيد للشرح ده:** الـ Verification Plan النهائي الموجود في فولدر `verification plan final/` (خمس شيتات HTML + ملف Excel `risc_v_verification_plan.xlsx`) — والمعمارية الأصلية `arcitecture/uvm_architecture_pro_updated.jpg`. أي حاجة مش فيهم، مش هنلاقيها في الشرح ده.

---

## جدول المحتويات

1. [إيه هو الـ Verification Plan أصلًا؟ وليه بنعمله؟](#1-إيه-هو-الـ-verification-plan-أصلا-وليه-بنعمله)
2. [الـ DUT بتاعنا بالتفصيل](#2-الـ-dut-بتاعنا-بالتفصيل)
3. [الـ Testbench Architecture الأصلية](#3-الـ-testbench-architecture-الأصلية)
4. [قاموس المصطلحات المهمة](#4-قاموس-المصطلحات-المهمة)
5. [الخريطة الكبيرة: من Requirement لـ Verification Closure](#5-الخريطة-الكبيرة-من-requirement-لـ-verification-closure)
6. [شيت System — شرح الـ 55 Requirement](#6-شيت-system--شرح-الـ-55-requirement)
7. [شيت Generation — مين بيولّد إيه وإزاي](#7-شيت-generation--مين-بيولّد-إيه-وإزاي)
8. [شيت Checking — مين بيفحص إيه وإزاي](#8-شيت-checking--مين-بيفحص-إيه-وإزاي)
9. [شيت Coverage — إزاي بنقيس ونقفل](#9-شيت-coverage--إزاي-بنقيس-ونقفل)
10. [شيت Test List — الـ 27 Test](#10-شيت-test-list--الـ-27-test)
11. [السيناريوهات والـ Corner Cases بالتفصيل](#11-السيناريوهات-والـ-corner-cases-بالتفصيل)
12. [Pass/Fail و Verification Closure](#12-passfail-و-verification-closure)
13. [جدول المسؤوليات النهائي](#13-جدول-المسؤوليات-النهائي)
14. [أسئلة بتتسأل كتير (FAQ)](#14-أسئلة-بتتسأل-كتير-faq)

---

## 1. إيه هو الـ Verification Plan أصلا؟ وليه بنعمله؟

ببساطة، الـ Verification Plan هو **الخطة اللي بتقولنا إحنا هنثبت إزاي إن الـ DUT بيعمل اللي المفروض يعمله** — قبل ما نقول "التصميم خلص".

تخيل إن معاك Core زي الـ CV32E40P: بيفتش تعليمات من ميموري، ينفّذها، يقرا ويكتب داتا. إزاي تثبت إن كل ده صح؟ مينفعش تقول "شغّلت برنامج وشكله تمام". لازم **قائمة مكتوبة** فيها:

- **إيه بالظبط** اللي المفروض التصميم يعمله (دي الـ Requirements — بتتاخد من الـ Databook ومواصفة RISC-V ومن قراءة الـ RTL نفسه).
- **إزاي هنجيب السيناريو** اللي يمارس السلوك ده (Generation).
- **إزاي هنعرف إنه اشتغل صح** (Checking).
- **وإزاي نعرف إننا جرّبنا كل الحالات** مش بس كام حالة (Coverage).

وليه بنعمله على الورق الأول؟ لأن من غيره هتلاقي نفسك بعد شهرين بتبني Testbench عشوائي، وفي الآخر **محدش يقدر يجاوب على سؤال: "هل احنا خلاص خلصنا؟"** الـ Plan هو اللي بيحدد "الخلاص" بشكل قابل للقياس — ده اللي بنسميه Verification Closure.

### علاقته بالـ RTL/DUT

كل Requirement في البلان مربوط بمصدر حقيقي: يا إما نص من الـ Databook (مثلاً فصل الـ Load-Store Unit)، يا إما مواصفة RISC-V (`standard/riscv-spec-20191213.pdf`)، يا إما سلوك متأكدين منه من كود الـ RTL نفسه (ولما ده يحصل بنسجّل مرجع الملف زي `cv32e40p_load_store_unit.sv`). **ممنوع يكون في البلان حاجة متخيلة** — لو حاجة مش معروفة، بتتسجّل كقيد أو ملاحظة بدل ما نخترع سلوك.

### علاقته بالـ Testbench Architecture

البلان مش معمول في الفراغ: كل Requirement في شيت System ليه عمود اسمه **component name** فيه المكوّنات اللي مسؤولة عنه في الـ Testbench (مين يولّد، مين يراقب، مين يفحص). فلو عايز تعرف "الفحص ده هيتعمل فين في الكود؟" — الجواب مكتوب في البلان نفسه، ومربوط بأسماء مكوّنات المعمارية الأصلية بالظبط (هنشوفها في الفصل 3).

> ✅ **الحكم النهائي على البلان:** لو كل Requirement اتحقق (Checks اتنفذت ومفيش غلطات) + كل Coverage اتقفلت → التصميم "verified". الـ Plan هو العقد.

### إزاي تقرأ ملفات البلان

| الملف | المحتوى | الصفوف |
|---|---|---|
| `System.html` | الـ Requirements نفسها (وصف + ملاحظة + component name) | 55 |
| `Generation_.html` | لكل Requirement: السيناريو بيتولّد إزاي + بأي sequence/object | 42 |
| `Checking_.html` | لكل Requirement: الفحص + اسم الـ property + مين بيفحص | 45 |
| `Coverage_.html` | لكل Requirement: إيه اللي لازم يتغطّى + اسم الـ covergroup | 40 |
| `test list.html` | الـ testbench tests اللي بتجمّع الكلام ده | 27 |
| `risc_v_verification_plan.xlsx` | نفس المحتوى في ملف Excel واحد | — |

**أهم حاجة تفهمها:** الـ ID واحد عبر كل الشيتات. `risc_lsu_04` هو نفس الـ Requirement في الأربع شيتات — مرة بيوصفه، مرة بيقول بيتولّد إزاي، مرة بيتفحص إزاي، مرة بيتغطّى إزاي. دي الـ **Traceability** — من غير ما تعمل ماتريكس خارجية.

---

## 2. الـ DUT بتاعنا بالتفصيل

### 2.1 مين هو الـ DUT؟

الـ DUT هو **`cv32e40p_top`** — المعالج CV32E40P بتاع OpenHW Group، بالـ parameters الافتراضية (من `parameters/parameters.png`):

```
COREV_PULP=0   COREV_CLUSTER=0   FPU=0   ZFINX=0   NUM_MHPMCOUNTERS=1
```

يعني عمليًا: **Core RV32IM نقي** — Base Integer (I) + Multiply/Divide (M). الـ Hardware نفسه فيه extensions تانية (C, Zicsr... إلخ) لكنها **بره هدف الـ verification** بتاعنا (Requirement `risc_sys_00` بيثبت ده رسميًا).

### 2.2 البورتات (Interfaces) بتاعته

| المجموعة | الإشارات | معناها |
|---|---|---|
| Clock/Reset | `clk_i`, `rst_ni` | كلاك + reset سالب async |
| Static config | `boot_addr_i`, `mtvec_addr_i`, `hart_id_i`, `dm_halt_addr_i`, `dm_exception_addr_i` | عناوين ثابتة بتتحط قبل التشغيل |
| **Instruction OBI** (master) | `instr_req_o`, `instr_gnt_i`, `instr_rvalid_i`, `instr_addr_o`, `instr_rdata_i` | بيفتش بيها التعليمات |
| **Data OBI** (master) | `data_req_o`, `data_gnt_i`, `data_rvalid_i`, `data_we_o`, `data_be_o`, `data_addr_o`, `data_wdata_o`, `data_rdata_i` | Load/Store |
| Tie-offs | `irq_i`, `debug_req_i`, `pulp_clock_en_i`, `scan_cg_en_i` | مدخلات لفيتشرز بره الهدف — بنثبّتها (شوف `risc_sys_06`) |
| Control | `fetch_enable_i` | الكور ميبدأش fetch غير لما تبقى 1 |
| Status (مستُهملة بحكم الـ scope) | `irq_ack_o`, `irq_id_o`, `debug_*_o`, `core_sleep_o` | مخرجات لفيتشرز بره الهدف |

### 2.3 بروتوكول OBI في 6 نقاط (لأن نص البلان مبني عليه)

1. **`req` يفضل عالي لحد ما ياخد `gnt`** — والعنوان (والداتا في الـ store) يفضلوا ثابتين طول المدة دي.
2. بعد الـ `gnt` يجي **`rvalid` لدورة واحدة بالظبط** لكل طلب — وبيحمل الـ read data (أو تأكيد الـ store).
3. **مفيش `rready`** — يعني الكور مش بيرجّع ضغط على الذاكرة وقت الاستجابة.
4. **مفيش transaction IDs** — فالردود لازم ترجع **in-order** (بنفس ترتيب الطلبات).
5. الـ Instruction fetch أقصاه **2 outstanding** (لأن `FIFO_DEPTH=2` في الـ prefetch buffer) — والـ Data side كمان لحد 2.
6. **مفيش error response أصلًا** في البروتوكول — يعني الذاكرة بتاعتنا دايمًا بتكمّل المعاملات بنجاح (مفيش error stimulus).

### 2.4 قوانين الـ Scope (أهم صفحة في الفصل ده)

`risc_scope_00` بيقولها صراحة: الفيتشرز دي **بره الهدف** — يا إما بنثبّتها tie-off يا إما منتولّدهاش أصلًا، و**ممنوع نبني ليها checkers أو coverage**:

> C-extension internals (aligner/compressed decoder) · CSR instructions/state · interrupts · debug · sleep unit/wfi/elw · FPU · CORE-V custom (PULP/CLUSTER) · atomics/PMP/user mode · performance counters · FENCE/ECALL/EBREAK/MRET.

وفي المقابل (`risc_sys_06`) عشان الكور يفضل شغال **طبيعي** لازم القيم دي تثبت طول السيميوليشن:

```
irq_i = 32'b0   debug_req_i = 1'b0   pulp_clock_en_i = 1'b0   scan_cg_en_i = 1'b0
```

يعني: مش بنختبر الفيتشرز دي، بس بنقودها قيادة صحيحة عشان متبوظش اللي بنختبره. دي قاعدة مهمة في أي مشروع: **"out of scope ≠ ignore" — يعني "drive to normal, but don't check".**

---

## 3. الـ Testbench Architecture الأصلية

دي المعمارية المعتمدة (`arcitecture/uvm_architecture_pro_updated.jpg`) — **اتأكدنا منها ضد الـ RTL وثبتناها زي ما هي** من غير أي إعادة تصميم:

```
                        env_cfg (Configuration)
                        Virtual Sequences
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│   v_sqr ────► Virtual Sequencer (ينسّق الـ sequencers بتاعة الـ agents)   │
│                                                                          │
│  ┌───────────────────────┐  ┌─────────────────────┐ ┌─────────────────┐  │
│  │ sys_ctrl_agent(Active)│  │ if_agent (Active)   │ │lsu_agent(Active)│  │
│  │  Sequencer            │  │  Sequencer          │ │  Sequencer      │  │
│  │  Driver  ────────────┐│  │  Driver (slave)     │ │  Driver (slave) │  │
│  │  Input Monitor       ││  │  Input Monitor      │ │  Input Monitor  │  │
│  └───────────────────────┘  └─────────────────────┘ └─────────────────┘  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────────────┐  │
│  │ if_passive_agent │  │lsu_passive_agent │  │ rvfi_monitor (Passive) │  │
│  │  Output Monitor  │  │  Output Monitor  │  │  Internal State Monitor│  │
│  └──────────────────┘  └──────────────────┘  └────────────────────────┘  │
│                                                                          │
│   ref_model (Subscriber) ──► uvm_scoreboard   Coverage (Subscriber)      │
│                                                                          │
├──────────────────────────────────────────────────────────────────────────┤
│  sys_ctrl_vif   if_vif   lsu_vif   rvfi_vif (internal probes)            │
├──────────────────────────────────────────────────────────────────────────┤
│            DUT: cv32e40p_top        (SVA module — bind cv32e40p_top)     │
└──────────────────────────────────────────────────────────────────────────┘
```

### 3.1 مين بيعمل إيه — سؤال وجواب

**"ليه الـ if_agent و lsu_agent Active؟"**
لأن الكور **master** على الاتنين OBI — فالـ Testbench لازم يلعب دور **الذاكرة (slave)**: يستقبل الطلب، يعمل `gnt`، وبعد شوية يرجّع `rvalid` + الداتا. جوه كل active agent في **memory model** (`instr_mem_model` جوه `if_agent` و `data_mem_model` جوه `lsu_agent`) — البرنامج بيتحمّل في الأول، والدرايفر بيرد منها. كمان في **environment guard assertions** جوه نفس الـ agents بتضمن إن الدرايفر بتاعنا نفسه مبيخالفش البروتوكول (مفيش `rvalid` من غير طلب، في `rvalid` واحدة بس لكل `gnt`...).

**"طيب ليه محتاجين الـ Monitors؟"**
لأننا محتاجين **نشوف الـ transactions اللي بتحصل فعليًا** على الـ interface ونستخدمها في الـ checking. في نوعين مراقبة في المعمارية:

- **Input Monitor** جوه الـ active agents — بيراقب نفس الـ interface اللي الدرايفر بيقودها (الطلبات والاستجابات من منظور الـ agent الشغال).
- **Output Monitor** جوه الـ **passive agents** (`if_passive_agent.output_monitor` و `lsu_passive_agent.output_monitor`) — passive بالكامل: بيسمبل الباص من بره من غير ما يقود حاجة، وبيطلّع **stream نظيف** للـ protocol SVA وللـ transaction logs اللي الـ scoreboard بيقارنها. معظم الفحوصات البروتوكولية في البلان (`SVA inside if_passive_agent.output_monitor`...) معمولة هنا لأنه المكان الطبيعي اللي بيشوف الباص كامل.

**"ومين بيفحص صحة النتايج نفسها؟"**
سلسلة التحقق الأساسية (هنشرحها بالتفصيل في فصل 8):

```
rvfi_monitor (الـ Internal State Monitor يلقط كل تعليمة بتتنفّذ فعلًا)
        │  retire stream: pc, instruction, rd, memory effects...
        ▼
ref_model (الـ golden RV32IM model بتاعنا — يحسب المفروض إيه يحصل)
        │  expected results
        ▼
uvm_scoreboard (يقارن expected vs actual + shadow/golden memory + bus logs)
        │  mismatch = FAIL ببلاغ فيه إيه الغلط وعند أي تعليمة
```

**"ليه في `rvfi_monitor` داخلي أصلًا؟ الـ Core مفيهوش RVFI port!"**
صح — دي نقطة تصميم واعية. الـ RTL مفيهوش RVFI interface جاهز، فالـ `rvfi_monitor` (اللي في الرسم اسمه **RVFI Monitor + Internal State Monitor**) بيبني "retire view" من **إشارات داخلية للكور** عبر `rvfi_vif` (internal probes) — نفس الأسلوب اللي الـ RTL نفسه بيستخدمه في فحوصاته الداخلية. ليه العناء ده؟ عشان المقارنة الصح تكون على **التعليمات اللي اتنفّذت فعلًا (committed)** مش اللي اتفتشت — لأن الـ prefetch بيجيب تعليمات كتير بتترمي (flushes، هنشوفها) ومش عايزين الـ fetch stream يبوظ الحسابات.

**"والـ env_cfg و v_sqr والـ Virtual Sequences؟"**
- `env_cfg` = الـ configuration object: كل الـ knobs (العناوين، سياسات الـ delays، أنماط الـ reset، seeds...).
- `v_sqr` (Virtual Sequencer) + **Virtual Sequences** = التنسيق فوق كل الـ sequencers: متى يحصل reset، متى يتغيّر نمط الـ delays، تزامن السيناريو كله.

### 3.2 الـ Interfaces

| الاسم | بيلف إيه |
|---|---|
| `sys_ctrl_vif` | clk/reset/config/tie-offs/fetch_enable |
| `if_vif` | إشارات الـ Instruction OBI |
| `lsu_vif` | إشارات الـ Data OBI |
| `rvfi_vif` | probes لإشارات داخلية (retire view) |

---

## 4. قاموس المصطلحات المهمة

| المصطلح | معناه ببساطة |
|---|---|
| **Requirement** | جملة واحدة: "التصميم المفروض يعمل كذا" — ليها ID وحالة verify |
| **Verification Item** | نفس الـ Requirement متقسم أفقيًا: Generation + Checking + Coverage |
| **Stimulus / Scenario** | الإدخالات اللي بنرميها على الـ DUT عشان نمارس السلوك |
| **Sequence** | كلاس UVM بيولّد سيناريو معيّن (برنامج تعليمات / نمط delays / reset) |
| **Transaction** | حدثة واحدة على باص (طلب fetch بعنوان X، رد ببيانات Y...) |
| **Agent** | حاوية: Sequencer + Driver + Monitor(s) لانترفيس واحد |
| **Active vs Passive** | Active بيقود إشارات (يدّي)، Passive بيراقب بس |
| **Driver (slave)** | هنا: بيرد على OBI كأنه ميموري — gnt/rvalid/data |
| **Monitor** | بيسمبل الإشارات ويطلّع transactions للباقي، من غير ما يقود |
| **Scoreboard** | المكان اللي بيتم فيه المقارنة expected vs actual |
| **Reference Model (ref_model)** | الموديل الذهبي بتاعنا: SV model بـ RV32IM يحسب النتيجة الصح |
| **Shadow / Golden Memory** | صورة متوقعة لمحتوى الميموري جوه الـ scoreboard للمقارنة |
| **Coverage (covergroup)** | عدّادات منظمة بتقول "الحالة دي حصلت؟" — coverpoint = حالة، cross = تركيبات |
| **SVA / assertion** | property بـ SystemVerilog بتتفحص كل كلاك؛ بتفشل فورًا لو اتحللت |
| **Bind** | توصيل module فيه assertions جوه hierarchy الـ DUT من غير ما نعدّل RTL |
| **Tie-off** | تثبيت مدخل على قيمة ثابتة طول السيميوليشن |
| **Outstanding** | طلبات اتحصل لها gnt ولسه مستنية rvalid |
| **In-order responses** | الردود بترجع بنفس ترتيب الطلبات (لأن مفيش IDs) |
| **Wait states** | دورات تأخير مقصودة على gnt/rvalid من الذاكرة |
| **Retire / Commit** | التعليمة "خلصت فعليًا" وأثّرت معماريًا (عكس اللي اتفتشت واترمى) |
| **Flush** | رمي تعليمات من الـ pipeline (مثلاً بعد taken branch) |
| **Hazard / Forwarding / Stall** | اعتماديات بين التعليمات: بيانات لسه متكتبتش → forward أو stall |
| **Misaligned access** | عنوان مش natural-aligned؛ الكور بيقسّمه لمعاملتين على الباص |
| **End-of-program sentinel** | عنوان معيّن بنعرف منه إن البرنامج خلص (ييجي في فصل 12) |
| **Traceability** | إن كل حاجة متتبّعة: نفس الـ ID من الـ requirement للـ coverage |
| **Verification Closure** | لما كل الـ checks عدّت + coverage اتقفلت → ممكن نوقّع |

---

## 5. الخريطة الكبيرة: من Requirement لـ Verification Closure

دي نفس السلسلة المطلوب شرحها، بس مطبّقة على مشروعنا الحقيقي:

```
Requirement (شيت System — مثلاً risc_lsu_04: الـ misaligned يتقسّم لمعاملتين)
    │  نفس الـ ID بينتشر أفقيًا
    ▼
Verification Item (Generation_ + Checking_ + Coverage_ لنفس الـ ID)
    │  Generation بيقول: الـ Stimulus بيتولّد إزاي
    ▼
Scenario / Stimulus
    ├─ riscv_program builder يولّد برنامج (32-bit RV32IM words) ─► instr_mem_model
    ├─ data pool ─► data_mem_model
    ├─ sys_ctrl_init_sequence ─► reset + boot_addr + mtvec + fetch_enable
    └─ obi_memory_sequence ─► gnt/rvalid delays على if_agent / lsu_agent
    │  عبر v_sqr / Virtual Sequences التنسيق، وenv_cfg تحمل الـ knobs
    ▼
DUT = cv32e40p_top (يفتش من الذاكرة، ينفّذ، يقرا/يكتب داتا)
    │
    ▼
Monitoring (ثلاث عيون):
    ├─ if_passive_agent.output_monitor  ─► IF OBI transaction stream + protocol SVA
    ├─ lsu_passive_agent.output_monitor ─► LSU OBI transaction stream + protocol SVA
    └─ rvfi_monitor ─► retire stream (committed فقط)
    ▼
Checking
    ├─ ref_model يحسب expected لكل تعليمة متقاعدة
    ├─ uvm_scoreboard يقارن expected vs actual (+ shadow/golden memory + bus logs)
    └─ SVA properties (جوه الـ output monitors / جوه الـ slave drivers) تلقط خروق فورًا
    ▼
Pass / Fail
    ├─ صفر mismatches + teardown: outstanding==0 + final GPR/mem == golden + مفيش timeout
    └─ أي mismatch/property fail = UVM_ERROR مبلّغ بالتعليمة والسياق
    ▼
Coverage (40 covergroup — coverpoints وcrosses من نفس الـ streams)
    ▼
Verification Closure = كل الـ Checks عدّت على كل الـ Tests/Seeds + Coverage مقفولة 100%
```

---

## 6. شيت System — شرح الـ 55 Requirement

الشيت متجمّع في 11 عيلة. هنمشي عيلة عيلة: **إيه المعنى → ليه مهم → مين مسؤول.**

### 6.1 عيلة `risc_sys_*` (00–07) — النظام والبورتات والـ scope

| ID | بيثبت إيه ببساطة | ليه مهم / إزاي بيتفحص |
|---|---|---|
| `risc_sys_00` | الـ DUT هو `cv32e40p_top` بالـ default parameters → ISA الفعلي RV32IM؛ والـ extensions التانية بره الهدف | عقد الهوية — لو حد غيّر parameter، كل البلان يتّعاد مراجعته. Verification top-level للكور كله (`top_tb`) |
| `risc_sys_01` | كل بورتات الـ top متوصولة ومرصودة — القايمة الكاملة مكتوبة | لو سلك مش مرصود، باغ محتمل مش هيترى. الربط عبر 4 interfaces: `sys_ctrl_vif`, `if_vif`, `lsu_vif`, `rvfi_vif` |
| `risc_sys_02` | `rst_ni` async active-low: طول ما هو 0 (ولحد `fetch_enable_i`) الكور ساكت: مفيش req على أي باص | أول عقد سلوكي. بيتفحص بـ SVA: `sys_reset_no_req_prop` جوه الـ passive output monitors + reset عشوائي التوقيت/المدة |
| `risc_sys_03` | أول fetch بعد الـ reset لازم يكون على `boot_addr_i` (word-aligned على الأقل) | لو الـ PC الأول غلط، كل حاجة بعدها غلط. SVA `sys_boot_addr_prop` + تعشيق `boot_addr_i` على نوافذ قانونية متنوعة |
| `risc_sys_04` | `mtvec_addr_i` بيعمل initialize للـ mtvec كـ `{mtvec_addr_i[31:8], 6'b0, 2'b01}` وأي exception بيروح للـ base `{mtvec[31:8], 8'h00}` (مش vectored — الـ interrupts بس اللي vectored) | دي قاعدة تصحّ أي "illegal instruction → فين يروح الـ fetch". متأكدين منها من `cv32e40p_if_stage.sv`. فحص عبر scoreboard + مراقبة الـ redirect |
| `risc_sys_05` | ممنوع أي `instr_req_o` قبل ما `fetch_enable_i` تبقى 1 (الكلاج الداخلي gated) | SVA `sys_no_fetch_before_fe_prop` + تأخيرات عشوائية لـ fe_delay (منها >8 دورات) |
| `risc_sys_06` | الـ tie-offs: `irq_i=0, debug_req_i=0, pulp_clock_en_i=0, scan_cg_en_i=0` طول السيميوليشن | "drive to normal only" — **مفيش checkers ولا coverage** على الفيتشرز دي |
| `risc_sys_07` | المخرجات الأساسية منهاش X/Z بعد الـ reset وهو شغال | SVA `$isunknown` جوه الـ passive output monitors — أول دفاع ضد باجز الـ uninit |

### 6.2 عيلة `risc_if_*` (00–09) — الـ Instruction Fetch

| ID | بيثبت إيه | التعليق التدريبي |
|---|---|---|
| `risc_if_00` | `instr_req_o` يفضل عالي ومستقر لحد `instr_gnt_i` | أشهر خرق OBI: تغيير العنوان وهو مستني. SVA جوه `if_passive_agent.output_monitor` (`if_obi_req_persistence_prop`, `if_obi_addr_stable_prop`) |
| `risc_if_01` | مفيش اعتماد combinational `rvalid→req`؛ و`gnt` ممكن تيجي قبل الطلب | بروتوكول OBI خام. بيتعمل له generation بـ `gnt_always_high` mode |
| `risc_if_02` | `rvalid` = دورة واحدة بالظبط لكل grant، in-order لتنين outstanding | الكور معتمد على الترتيب لأن مفيش IDs → العدّادات + env assertions جوه `if_agent` |
| `risc_if_03` | back-to-back fetch ممكن من غير فجوات (req متكرر كل دورة) | بيستخلص أعلى throughput؛ coverage منفصل (`if_obi_back2back_cg`) |
| `risc_if_04` | الـ prefetch بيجيب لحد DEPTH=2 كلمة زيادة عن كودك — والميموري لازم يكون من غير side effects | **قاعدة ذهبية**: الذاكرة متفترضش fetch==execute — أساس كل منطق الـ retire بعدين |
| `risc_if_05` | Jump (JAL/JALR بيتحسب في ID) → flush للـ prefetch والـ IF، والـ fetch يعيد من الهدف | التعليمات اللي اتفتشت ورا القافز **لازم تتشال من الـ scoreboard** — projects كتير بتوقع هنا |
| `risc_if_06` | نفس القصة للـ taken branch — بس بيتحسم في **EX** (فترة الـ speculative window أطول) | الـ scoreboard يقارن PC stream: اللي اتفتشت fall-through مش المفروض يرتكّب أثر |
| `risc_if_07` | الـ illegal → flush + fetch يعيد من **mtvec base** | يربط الـ decoder بالـ control flow — شرط أساسي لسيناريوهات الـ illegal |
| `risc_if_08` | **قيد توليد**: كل البرامج 32-bit فقط، word-aligned، bounded، وتنتهي عند **end-of-program sentinel address** الـ teardown بيعرفه | عشان نبعد عن الـ aligner/compressed decoder (C — بره الهدف) — لكنها بتتمر كـ pass-through برضه |
| `risc_if_09` | مفيش instruction-address-misaligned exception في الكور ده؛ أي trap على stream نضيف = bug | **anti-check** — قفل باب الغلطات التانية |

### 6.3 عيلة `risc_lsu_*` (00–09) — الـ Load/Store

| ID | بيثبت إيه | التعليق التدريبي |
|---|---|---|
| `risc_lsu_00/01` | نفس قواعد الـ IF على الـ Data side (استقرار الطلب، دورة rvalid واحدة، in-order ≤2 outstanding) | ضبطت من الـ databook فصل LSU |
| `risc_lsu_02` | معلومة توقيت: بدون wait states الـ load/store معاملة واحدة (EX+WB)، والـ misaligned معاملتين | informational — بتتقفل من latency coverage |
| `risc_lsu_03` | جدول الـ **byte-enable**: SW→1111، SH→0011/0110/1100 (وفي القطع: 1000 ثم 0001)، SB→0001/0010/0100/1000، سو misaligned (1110/1100/1000 + المكمّلات) | متاخد من `cv32e40p_load_store_unit.sv` — الـ BE غلط = بايتس متكتبة غلط بصمت. SVA/LUT جوه الـ output monitor: **بس الأنماط دي تظهر** (`lsu_be_legal_prop`) |
| `risc_lsu_04` | **Misaligned = معاملتين (الأدنى أولًا)، مفيش exception أبدًا** | أحد أهم سلوكيات الكور — فحص مزدوج: ترتيب SVA (`lsu_misaligned_order_prop`) + scoreboard يجمع الجزئين ويقارن النتيجة (`sb_load_result_match`, `sb_no_exc_on_misaligned`) |
| `risc_lsu_05` | تدوير `data_wdata_o` حسب `addr[1:0]` (بت الـ rtl formula) | الـ store بتاع byte بيروح lane صحيح بس — مقارنة نهائية للذاكرة (`sb_store_mem_match`) |
| `risc_lsu_06` | LB/LH sign-extend، LBU/LHU zero-extend | كلاسيك غلطات sign extension — golden compare لكل retired load |
| `risc_lsu_07` | الـ load يشوف آخر store لنفس العنوان (in-order + single hart) | الـ scoreboard shadow memory بتفرض دي كل مرة |
| `risc_lsu_08` | التعليمات اللي اتـflush ماتعملش side effects على الـ data bus | عمليًا: bus stream يطابق **بالظبط** اللي retire فقط (`sb_lsu_trans_stream_match`) |
| `risc_lsu_09` | مفيش error response في الـ Data OBI | الـ env ما يتوقعش ولا يولّد errors |

### 6.4 عيلة `risc_dec_*` (00–04) — الـ Decode والـ ISA

| ID | بيثبت إيه |
|---|---|
| `risc_dec_00/01` | كل RV32I + الـ 8 بتاعة M قانونية وبصح (`sb_rv32i_result_match` / `sb_rv32m_result_match` عبر golden compare) — أساس كل حاجة |
| `risc_dec_02` | كاشف الـ illegal جوه مساحة الـ RV32I/M بس — 3 فئات: **(a)** major opcode غير معرّف، **(b)** opcode سليم + funct3/funct7 غير معرّفين، **(c)** shift encodings محجوزة (funct3=001/101 مع funct7 مش 0000000/0100000) | كل واحدة لازم تترصد → trap → متتكتبش rd وماتطلقش memory request |
| `risc_dec_03` | منطق الـ x0: تعليمات بـ rd=x0 أو مصادر x0 **قانونية** عادي؛ بس الـ encodings الغير معرّفة تفضل illegal حتى لو فيها x0 | بيميّز بين "legal بدون أثر" و"illegal واضح" |
| `risc_dec_04` | قيد توليد: FENCE/FENCE.I/ECALL/EBREAK/WFI/MRET + كل الـ CSR + كل الأوبكودت الخارجة (F/A/custom) **منتولّدهاش أصلًا** | "constraint only" — مفيش check/coverage لأنهم بره الهدف |

### 6.5 عائلات الحساب: `risc_alu_*` و `risc_m_*` (00–04/00–02)

- **`risc_alu_00`**: كل عمليات الـ ALU بنتيجة صحيحة — golden compare مع chains اعتمادية (ده اللي بيختبر الـ forwarding نفسه).
- **`risc_alu_01`**: الـ barrel shifter single-cycle والـ shamt = 5 bits (`sb_shift_result_match` على كل أركان الـ shamt).
- **`risc_alu_02`**: LUI/AUIPC — وAUIPC position-dependent؛ بيتجرب على PC's متنوعة (`sb_upper_result_match`).
- **`risc_m_00`**: MUL في دورة واحدة (multiplier بتاعه محترم).
- **`risc_m_01`**: MULH\* بـ 5 دورات FSM (STEP0→FINISH) — والـ EX بيقف؛ كمان: **مفيش spurious write-back في فترة الـ stall** (`sb_no_spurious_wb`).
- **`risc_m_02`**: DIV/REM متكرر بـ early termination — latency 3..35 دورة على حسب leading zeros للمقسوم عليه؛ بنقيس **التوقيت نفسه** (`sb_div_latency_range`) مش بس النتيجة.
- **`risc_m_03`**: قيم الـ RISC-V الرسمية للحالات الخاصة: div-by-zero → quotient=-1، remainder=dividend؛ overflow (INT_MIN/-1) → quotient=INT_MIN، remainder=0 (`sb_div_corner_results`).
- **`risc_m_04`**: multicycle أقدم من taken branch **يكمّل ويكتب طبيعي** (مش يتقتل) — سيناريو مشهور صعب (`sb_multicycle_plus_branch`).

### 6.6 عائلات الـ pipeline: `risc_haz_*` و `risc_rf_*` و `risc_rst_*`

| ID | بيثبت إيه |
|---|---|
| `risc_haz_00` | RAW forwarding من EX لـ ID يخلّي الـ dependent يلحق بدون stall (0-cycle gap بين ALU producer وconsumer) — متصحّح من muxes الـ ID |
| `risc_haz_01` | **Load-use: stall دورة واحدة بالظبط** — وبس على مسافة 1؛ من مسافة ≥2 مفيش stall (`sb_load_use_1cycle`) |
| `risc_haz_02` | JALR اللي rs1 بتوعه جاي من instruction قبله مباشرة → `jr_stall` دورة |
| `risc_haz_03` | **WAW collision** (load في WB + ALU/MUL/DIV أحدث في EX نفس الـ rd): القيمة النهائية = قيمة **الأحدث** (`sb_waw_younger_wins`) — corner مشهور تاني |
| `risc_haz_04` | **Cleanliness للـ flush**: أصغر من taken branch/jump/trap → صفر آثار معمارية |
| `risc_rf_00` | x0 صلب للصفر: قراءة دايمًا 0، والكتابة تتجاهل — حتى لو load رايح x0 |
| `risc_rf_01` | x1..x31 موجودين (الـ FF RF بـ reset=0) — وده فحص مرة واحدة في أول test بس: **خاصية implementation مش ISA** (مسجّلة كده بصراحة) |
| `risc_rst_00/01/02` | الـ reset أثناء: fetch outstanding / data outstanding (منها نص misaligned pair) / DIV-MULH + branch window — بعده re-boot نظيف، مفيش rvalid قديم يتاكل، مفيش write-back من تعليمة اتقتلت (`sb_reset_resync`) |

### 6.7 عيلة البيئة والـ scope: `risc_env_*` و `risc_scope_00`

- **`risc_env_00`**: الـ TB فيه الـ memory models (عرفناها)، البرنامج والـ data pool بيتحملوا قبل التشغيل، وكل الكلمات **deterministically initialized** → أي load من مكان متكتبش، reproducible ويطابق الـ golden.
- **`risc_env_01`**: الخصائص بتاعة الـ OBI slaves: gnt delay 0..K، rvalid delay 1..L، modes (zero-wait / random / long-stall / gnt-always-high)، وضغط متوازي على الباصين.
- **`risc_env_02`**: الـ **teardown**: آخر الـ test تتعهد كل الـ outstanding → الـ retire stream complete → مقارنة نهائية GPR + dmem مع الجولدن → **watchdog** لو الكور علّق.
- **`risc_scope_00`**: (شرحناها في فصل 2.4) — قائمة "out-of-goal" مسجلة كقيود بس.

---

## 7. شيت Generation — مين بيولّد إيه وإزاي

### 7.1 نموذج التوليد الأساسي (افهمها كويس)

```
┌─ riscv_program (program builder object) ─────────────────────────────┐
│  بيبني البرنامج كلمات 32-bit: RV32IM legal إجباري + word-aligned     │
│  targets + bounded + sentinel للنهاية + constraints زي:              │
│  مفيش F/A/custom/CSR أوبكودز (risc_dec_04)، RVC ممنوع (risc_if_08)   │
└──────────────┬───────────────────────────────────────────┬───────────┘
               │ encodes words                               │ data pool
               ▼                                           ▼
     instr_mem_model (if_agent)              data_mem_model (lsu_agent)
               ▲                                           ▲
     الكور يفتش طبيعي                            طلبات load/store عادية
```

**ليه نولّد البرنامج كامل الأول بدل ما نرد على كل fetch لحظيًا (reactive)?**
أسلوبين معتمدين — احنا اخترنا التوليد المسبق لأنه: (1) كل run قابلة لإعادة الإنتاج بالـ seed؛ (2) الـ golden model يقدر يحسب الـ expected بسهولة لأنه بيقرا نفس الصورة؛ (3) الـ coverage attribution نضيف. الـ reactive متبقى بس في **تشكيل توقيت** الردود (delays) — مش تعليمات جديدة.

### 7.2 أنواع الـ sequences في البلان

| العائلة | الـ sequences | وظيفتها |
|---|---|---|
| System | `sys_ctrl_init_sequence`, `sys_ctrl_reset_inject_sequence` | reset (عشوائي/حقن في التوقيت)، boot/mtvec/fe delays |
| OBI timing | `obi_memory_sequence` (قناة IF وقناة LSU، modes: zero-wait / random / long-stall / gnt_always_high) | تشكيل gnt/rvalid delays — الضغط التوقيتي |
| برامج موجّهة | `riscv_alu_sequence`, `riscv_upper_sequence`, `riscv_shift_sequence`, `riscv_lsu_sequence`, `riscv_lsu_misaligned_sequence`, `riscv_mem_b2b_sequence`, `riscv_branch_sequence`, `riscv_mul_sequence`, `riscv_div_sequence`, `riscv_div_corner_sequence`, `riscv_illegal_sequence`, `riscv_hazard_sequence`, `riscv_waw_sequence`, `riscv_ctrl_mix_sequence`, `riscv_corner_case_sequence` | كل واحدة بلّغت لعيلة requirements معيّنة (شوف عمود component/Object name في الشيت) |
| عشوائي | `riscv_random_mix_sequence` | خليط كامل RV32IM بنسب أوزان + نسبة illegals ~2% + delays عشوائية |

### 7.3 لمحات Generation مهمة (من الصفوف نفسها)

- **`risc_if_08`**: القيود دي **متطبّقة داخل الـ program builder** مش كلام: عناوين القفزات word-aligned وجوّه حدود البرنامج؛ مفيش self-modifying code.
- **`risc_lsu_03/04`**: sweeps رسمية: lw/sw على الأوفستات الأربعة، lh/sh على الأوفستات، lb'/sb' على الأوفستات + أزواج misaligned متتالية.
- **`risc_lsu_05/06`**: حمامات داتا موجّهة (`0x80000000, 0xDEADBEEF, 0xAAAAAAAA...× كل الأوفستات`)؛ وتحميلات الـ MSB=0/1 في كل lane للـ sign-extension.
- **`risc_m_02/03`**: تفريق الـ divisor على bins الـ leading zeros عشان نغطّي الطيف 3..35؛ + مصفوفة أركان موجّهة (dividend × divisor {0, 1, -1, INT_MIN, INT_MAX}).
- **`risc_haz_00..03`**: chains اعتمادية كثيفة بمسافات 1/2/3 (وأطول) على rs1/rs2/الاتنين + producer→JALR pairs + collisions لـ WAW.
- **`risc_rst_00..02`**: **حقن الـ reset في توقيتات خطرة**: خلال IF/LSU wait states، نص misaligned pair، أثناء DIV/MULH، بعد taken branch مباشرة.
- **`risc_dec_02`**: items من كل فئة غير قانونية (a)–(c) بفلر عشوائي + حقن في مواضع عشوائية من الـ stream.

---

## 8. شيت Checking — مين بيفحص إيه وإزاي

البلان فيه **4 أسلحة فحص** — كل واحد مكانه الصح:

### السلاح 1 — Golden End-to-End Compare (الأساسي)

> كل تعليمة بتتقاعد → الـ `rvfi_monitor` يبلغها → الـ `ref_model` يحسب المفروض يحصل → الـ `uvm_scoreboard` يقارن.

انضباط تام لأنه in-order single-issue؛ والمقارنة شاملة: **قيمة rd · تأثيرات الذاكرة · الـ PC الجاي**. بصراحة: لو فيه باغ منطقي في أي مرحلة (ALU, forwarding, control...) هيبان هنا — بأول تعليمة تختلف، ببلاغ فيه expected vs actual.

### السلاح 2 — Protocol SVA في الـ passive output monitors

الفحوصات دي بتشتغل كل كلاك على الباص مباشرة — بتلقط الخرق لحظته:

| Property | بيفحص إيه |
|---|---|
| `sys_reset_no_req_prop` | مفيش req أثناء/بعد reset لحد fetch_enable |
| `sys_boot_addr_prop` / `exc_trap_base_prop` | أول fetch = boot؛ redirect على mtvec base بعد illegal |
| `sys_no_fetch_before_fe_prop` | مفيش fetch قبل fetch_enable |
| `outputs_no_x_prop` | `$isunknown` على المخرجات |
| `if_obi_req_persistence_prop` / `if_obi_addr_stable_prop` (+ نفسوا `lsu_*`) | ثبات الطلب أثناء انتظار الـ gnt |
| `lsu_be_legal_prop` | الـ BE pattern دايمًا من الجدول القانوني |
| `lsu_misaligned_order_prop` | المعاملتين بالترتيب والـ BEs المكمّلة الصح |
| `if_obi_trans_match_prop` | عدّادات: #(req&&gnt) == #(rvalid) بنهاية الـ test |

### السلاح 3 — Environment Guard Assertions جوه الـ slave drivers

دي بتفحص **البيئة نفسها** (عشان البيئة متبقاش مصدر غلط): `if_obi_slave_protocol_prop`, `lsu_obi_slave_protocol_prop`, `lsu_max_outstanding_prop`, `env_slave_protocol_prop` — أبدًا منشوف `rvalid` من غير outstanding، أبدًا `rvalid`زيادة لنفس الـ gnt، الـ rdata ثابت وقت الشفة.

### السلاح 4 — Scoreboard Deep Checks (logic مش مجرد compare)

| Property | الحكاية |
|---|---|
| `sb_fetch_stream_consistent` | fetch log ↔ retire stream — مفيش كلمة اتكررت/اتفقدت |
| `sb_pc_flow_match`, `sb_jump_link_match` | الـ PC stream يمشي زي ما الـ branch/jump/sequential rules بتقول؛ وقيم الـ link صح |
| `sb_flush_clean` | fall-through بعد taken branch مش المفروض يظهر له أثر |
| `sb_trap_redirect` / `sb_no_unexpected_trap` / `sb_illegal_trap` | الـ trap بيحصل بس عند الـ illegal، والهدف base، ومن غير rd/mem write |
| `sb_lsu_trans_stream_match` | معاملات الـ data bus تطابق **بالظبط** اللي retire (مش أكتر ولا أقل) |
| `sb_mem_consistency`, `sb_store_mem_match`, `sb_load_result_match` | الذاكرة (shadow/golden) متطابقة byte-by-byte؛ sign-extension صح |
| `sb_load_use_1cycle`, `sb_div_latency_range` | فحوصات **توقيت**: stall دورة واحدة؛ latency جوّه 3..35 |
| `sb_waw_younger_wins`, `sb_multicycle_plus_branch`, `sb_no_spurious_wb` | الـ corner semantics |
| `sb_x0_zero`, `sb_rf_reset_zero_opt` | x0 دايمًا صفر؛ (والأخير فحص اختياري موثّق implementation-specific) |
| `sb_reset_resync` | أول fetch بعد reset = boot وصفر mismatches بعدها |
| `sb_final_state_match`, `env_teardown_check` | الإغلاق النهائي (هنرجع له في فصل 12) |

> **قاعدة للتعلم:** أي فحص عنده "وقت حدوث" معروف على الباص → SVA في الـ monitor. أي فحص عتقده "منطقة النتيجة" → scoreboard/golden. أي فحص عن **البيئة** نفسها → assertions جوه الـ agents.

---

## 9. شيت Coverage — إزاي بنقيس ونقفل

الـ Checking بيجاوب "صح ولا غلط؟" — الـ Coverage بيجاوب **"وصلنا لكل الحالات ولا لسه؟"**. في البلان 40 صف، كل واحد بيربط Requirement بـ **covergroup** باسم واضح. أمثلة حسب النوع:

**Protocol coverage (على الباص):**
`if_obi_wait_states_cg` (bins: 0, 1, 2, 3-7, >7)، `if_obi_outstanding_cg` (1 و 2)، `if_obi_rvalid_latency_cg`، `if_obi_back2back_cg`، `if_gnt_before_req_cg` — ومثيلاتها `lsu_*` (`lsu_stall_mid_pair_cg` بيغطّي stall نص misaligned pair) و`dual_bus_stress_cg` (ضغط متوازي).

**ISA coverage:**
`rv32i_instr_cg` (كل mnemonic مرة على الأقل — ADD/SUB وSRL/SRA متمايزين بـ funct7)، `rv32m_instr_cg`، `m_operands_cg`، `alu_operands_cg` (أركان: 0, 1, -1, INT_MIN, INT_MAX، أنماط متناوبة)، `shift_amount_cg` ({0,1,15,16,31}×op×MSB)، `upper_imm_cg`، `reg_access_cg` (الـ 32 سجل كـ rd/rs1/rs2)، `x0_usage_cg`.

**Scenario coverage:**
`branch_taken_cg` + `branch_offset_cg` + `branch_operands_cg` (taken/not × اتجاه × علاقة مقارنة)، `jump_cg`، `trap_context_cg`، `illegal_category_cg` + `illegal_context_cg` (الفئات (a)–(c) × back-to-back × أثناء wait-states)، `misaligned_wait_cross_cg` (misaligned × wait على beat1/beat2)، `mem_b2b_cg` (st→ld نفس العنوان/تداخل lanes...)، `load_use_cg`، `raw_distance_cg`، `jr_hazard_cg`، `waw_cg`، `multicycle_branch_cg`، `div_latency_cg` (bins 3 / 4-10 / 11-20 / 21-34 / 35).

**Reset/Boot coverage:**
`reset_context_cg` (reset في: خامل / IF outstanding / LSU outstanding / multicycle / branch window)، `boot_addr_cg` (أركان النافذة + walking-1 على العناوين العليا)، `mtvec_base_cg`، `fetch_enable_delay_cg` (0 / 1 / 2-8 / >8).

### القواعد الذهبية للـ coverage هنا

1. **متغطّاش الـ fetched stream — غطّي الـ retire stream**: لأن الـ prefetch بيجيب حاجات متترميش؛ لو سمبلت من الـ fetch هتشوف تعليمات "وهمية" في الـ coverage.
2. **ممنوع bins لحاجات بره الهدف** — ولا حتى "مغطاة بس مقفولة": لو بره الـ scope → مش موجود (قرار `risc_scope_00`).
3. الـ cross coverage مكانّه في السيناريوهات اللي معناها تفاعل (misaligned × wait، branch × producer، reset × context...).

---

## 10. شيت Test List — الـ 27 Test

الـ test = sequence (أو وضع منها) + knobs + seed. هتلاحظ إن كل test مربوطة بعيلة requirements:

| الفئة | الـ tests | الهدف |
|---|---|---|
| System/Boot | `risc_reset_test`, `risc_fetch_enable_stress_test` | أركان الـ boot/fe مع sweeps |
| ALU | `risc_alu_r_type_test`, `risc_alu_i_type_test`, `risc_shift_test`, `risc_upper_test` | أنواع العمليات + immediates أركان |
| LSU | `risc_load_store_aligned_test`, `risc_load_sign_test`, `risc_misaligned_test`, `risc_mem_b2b_test` | offsets/corners/sign/split/consistency |
| Control | `risc_branch_test`, `risc_jump_test`, `risc_jalr_hazard_test` | كل الشروط + الاتجاهات + hazards الـ rs1 |
| M | `risc_mul_test`, `risc_div_test`, `risc_div_corner_test` | sign-matrix + طيف الـ latency + كل الأركان |
| Negative | `risc_illegal_instr_test`, `risc_illegal_context_test` | الفئات (a)–(c) + سياقات صعبة |
| Hazards | `risc_hazard_raw_test`, `risc_load_use_test`, `risc_waw_test`, `risc_multicycle_branch_test` | chains/مسافات/collisions/flush متداخلة |
| Timing stress | `risc_obi_waitstate_test`, `risc_obi_stall_stress_test`, `risc_gnt_always_high_test` | delays عشوائية/تشبّع الـ 2-outstanding/gnt مفتوحة |
| Reset | `risc_reset_inject_test` | حقن في كل السياقات الخطرة |
| Mix | `risc_random_mix_test` | streams عشوائية كاملة (و illegals ~2%) + delays عشوائية |

**متعة الجونيور:** لما برنامج يكون **bounded وينتهي عند sentinel address** — فنهاية الـ test معلومة سلفًا، ولسه الـ teardown هيقفل الحساب (فصل 12).

---

## 11. السيناريوهات والـ Corner Cases بالتفصيل

دي اللي بتفرق بين خطة سطحية وخطة مهندس — دي اللي الـ reviewers بيدوروا عليها:

1. **Misaligned split** (`risc_lsu_04`): المعاملة بتتقسم لاتنين بترتيب محدد مع BEs مكمّلة — والتحقق مزدوج: ordering على الباص + نتيجة مجمّعة صح. **ليه صعب؟** لأن الباغ ممكن يكون في ترتيب، lane، أو دمج — كل واحد ليه فحص.
2. **Load-use دورة واحدة بالظبط** (`risc_haz_01`): لو الكور عمل stall زيادة أو ناقص → باغ أداء أو منطق؛ بنقيسه من retire spacing عند صفر wait.
3. **WAW collision — الأحدث يكسب** (`risc_haz_03`): load في WB + ALU أحدث في EX على نفس rd — القيمة النهائية لازم تكون بتاعة الأحدث وفق الـ ISA (وهنا بييجي دور الـ golden final compare).
4. **Multicycle + Branch** (`risc_m_04`): الـ DIV/الـ MULH الأقدم يكمّل ويكتب عادي حتى لو branch أحدث اتاخد — **مش** يترمي مع الـ flush.
5. **Flush cleanliness** (`risc_haz_04`/`risc_if_06`/`risc_lsu_08`): ثلاث شيكات مترابطة (PC stream, register effects, bus stream) عشان نمسك أي أثر زاحف من الطريق المرمي.
6. **Reset أثناء كل حاجة** (`risc_rst_00–02`): مش reset في البداية بس — حقن أثناء outstanding، نص pair، أثناء DIV، بعد branch — وبعدها re-boot نظيف وملف السجل sync من الأول.
7. **تشبّع الـ outstanding** (`risc_obi_stall_stress_test`): ضغط طويل في windows mem-heavy عشان الباصين يوصلوا 2-outstanding — والـ coverage لازم تشوفها.
8. **gnt دايمًا عالية** (`risc_gnt_always_high_test`): الـ slave مفتوح حتى لو مفيش طلب — بيكشف assumptions خفية في منطق الباص.
9. **الـ illegal contexts** (`risc_illegal_context_test`): back-to-back، أثناء IF stall، بعد taken branch — لأن الـ controller واخد قرارات كتير وقتها.
10. **DIV أركان**: div-by-zero + overflow + dividend=0 + الطيف 3..35 — كل واحدة مقفولة بمصفوفة موجّهة مش حظ.

---

## 12. Pass/Fail و Verification Closure

**إمتى الـ test "ناجح"؟** (من `risc_env_02` + checks الإغلاق):

1. البرنامج وصل النهاية بتاعته (end-of-program **sentinel address** معروف للـ teardown).
2. الذاكرة سلّمت **كل** الـ outstanding (drain) → صفر pending على الباصين.
3. الـ retire stream **مكتمل** ومطابق للـ expected.
4. **مقارنة النهاية**: final GPR state == golden، final dmem == golden (`sb_final_state_match`).
5. صفر UVM_ERRORs / property failures طول الرحلة + الـ **watchdog** ما ضربش.

**ولو فشل؟** أول mismatch/timeout → خطأ مبلّغ فيه: رقم/وصف التعليمة، expected vs actual، وحالة الـ outstanding — عشان الساعة 3 الفجر تعرف المشكلة فين (الـ logs بتاعة الـ monitor/الـ scoreboard هي خريطة الإنقاذ).

**Closure للريليس =** كل الـ 27 test × seeds خضراء + صفوف الـ coverage الأربعين بكل الـ coverpoints وcrosses المطلوبة مقفولة + مفيش requirement شكلها verified على ورق بس.

---

## 13. جدول المسؤوليات النهائي

| المكوّن (بالاسم النهائي) | مسؤوليته في البلان |
|---|---|
| `top_tb` | الربط: يبني الـ DUT والـ interfaces، يوزّع الـ virtual interfaces، ويشارك في الـ teardown |
| `sys_ctrl_agent` (Sequencer/Driver/Input Monitor) | ينفّذ reset + config + tie-offs + fetch_enable + حقن الـ reset |
| `if_agent` (Sequencer/Driver/Input Monitor + `instr_mem_model`) | slave للـ IF OBI: يرد من البرنامج + يشكّل delays + env guard assertions |
| `lsu_agent` (نفس التركيب + `data_mem_model`) | slave للـ Data OBI: loads/stores + BE semantics + delays |
| `if_passive_agent.output_monitor` | مراقبة سلبية للـ IF bus: transactions log + protocol SVA (boot/reset/BE-free props...) |
| `lsu_passive_agent.output_monitor` | مراقبة سلبية للـ Data bus: ordering/BE props/عدّادات/logs للـ scoreboard |
| `rvfi_monitor` | بناء الـ retire stream (الملتزم فقط) من الإشارات الداخلية |
| `ref_model` | الـ golden RV32IM model: نتائج، PC، تأثيرات ذاكرة، توقيع trap |
| `uvm_scoreboard` | المقارنة + shadow/golden memory + bus-vs-retire match + الإغلاق النهائي |
| الـ slave drivers جوه الـ agents | env guard assertions (بروتوكول الردود) |
| Coverage (Subscriber) | تجمع الـ covergroups من الـ streams (الـ committed أساسًا) |
| `v_sqr` + Virtual Sequences + `env_cfg` | التنسيق والـ knobs (مش بيفحصوا حاجة بأنفسهم) |
| `riscv_program` (builder) | يفرض قيود الـ generation (32-bit/aligned/bounded/صيغ قانونية بس) |

---

## 14. أسئلة بتتسأل كتير (FAQ)

**س: ليه الـ monitors اتقسمت active input monitor و passive output monitor؟**
ج: عشان الملكية واضحة: الـ input monitor بيراقب من منظور الـ agent الشغال (وهو بيقود)، والـ passive agent مكانه الوحيد اللي **ما بيقودش** فيه حاجة — فـstreamه نضيف ومناسب يبقى مصدر الفحوصات البروتوكولية والـ logs الرسمية. التنين بيسمبلوا نفس الإشارات، بس بأدوار مختلفة تمامًا.

**س: ليه مش بنعمل check للـ interrupts/debug/CSR مع إن البورتات موجودة؟**
ج: لأنهم بره هدف الـ verification. القرار موثّق (`risc_sys_06`/`risc_scope_00`): بنثبّت المدخلات عشان الكور يشتغل طبيعي — و**منمنع** بناء checkers/coverage ليهم عشان منضيعش مجهود في مساحة مش مطلوبة.

**س: إزاي المقارنة بين fetched و retired مش بتبوظ؟**
ج: ما الـ golden model مش بيمشي على الـ fetched أصلًا — الـ `ref_model` بيستيب على الـ **retire stream** بس. الـ fetched بيتفحص بروتوكوليًا (streams/counters)، مش دلالاتيًا.

**س: الـ misaligned المفروض exception ولا لأ؟**
ج: **لا — أبدًا** في الكور ده (`risc_lsu_04`): بيتقسم لمعاملتين وبيكمّل عادي. أي tool بيقولك "misaligned exception" هنا، بيسلّم bug.

**س: إيه الفرق بين SVA و scoreboard check؟**
ج: SVA = لقط فوري كل كلاك على إشارات (بروتوكول/توقيت/ثبات). Scoreboard = حكم منطقي على معنى (قيم، ترتيب الـ retire، محتوى ذاكرة). الاتنين مكملين.

**س: ليه الـ coverage من الـ retire stream؟**
ج: عشان الـ prefetch speculative — تغطية الـ fetched هتنفخ الـ numbers بتعليمات متنفذتش.

**س: هو الـ plan فيه 55 requirement — كفاية؟**
ج: السؤال الصح: كل سلوك قابل للملاحظة من الـ databook/RTL جوه الـ scope متغطّى بـ requirement ليه generation+check+coverage؟ — أيوه، ومصفوفة الـ traceability هي اللي بتثبت ده بنفسها (الـ ID نفسو).

---

> **آخر كلمة للجونيور:** الـ Verification Plan مش فورماليتي — هو **العقد اللي بنحاسب بيه نفسنا** قبل ما نحاسب التصميم. لو قريته وفهمت كل requirement مين بيولّده ومين بيفحصه ومين بيغطّيه — يبقى فهمت المشروع كله.
>
> *المصدر:* `verification plan final/*.html` + `risc_v_verification_plan.xlsx` — بنفس الـ IDs والأسماء. أي اختلاف بين الشرح ده والشيتات، الشيتات هي المرجع.
