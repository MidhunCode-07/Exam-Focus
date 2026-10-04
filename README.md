# Exam Focus Project Document

**Project:** Exam Focus  
**Document type:** Project overview and technical handover  
**Prepared by:** Manus AI  
**Current project path:** `/home/ubuntu/exam-focus`  
**Current saved checkpoint:** `e1d9dc58`

## 1. Project summary

Exam Focus is a cross-platform study-support application for Android and web. Its central idea is **“First study, then earn screen time.”** The application helps students prepare for examinations by combining focused study sessions, short assessments, score-based rewards, timetable planning, an AI study assistant, and distraction-control workflows.

The intended loop is:

> **Study → Test → Score → Reward**

A student studies a selected topic, completes a quiz or mock test, and receives limited entertainment screen time after achieving the configured passing score. The default product requirement is that a score above 80% unlocks approximately 15–30 minutes of reward time. Leaving an active quiz early triggers a 30-minute penalty during which reward access remains unavailable.

The current codebase is an MVP. The web and Expo application flows are implemented, the backend is integrated, and the AI assistant now uses the server-side Forge LLM for dynamic responses. Native Android device-level blocking is implemented as an optional custom native module, but it requires an Android development build rather than standard Expo Go.

## 2. Objectives and user requirements

Exam Focus was developed to address the following student needs:

1. Students can create an exam-focused plan instead of manually managing several unrelated productivity tools.
2. Students can select topics from multiple education pathways, including school, intermediate, B.Tech, medical, commerce, humanities, computer science, and competitive-exam preparation.
3. Students can upload a timetable rather than typing every examination date manually.
4. The timetable parser can use AI to propose examination dates for user confirmation.
5. Students can ask an integrated AI assistant for study guidance and doubt clarification.
6. The assistant must be unavailable during a locked quiz so that the assessment remains meaningful.
7. Abandoning a quiz activates a 30-minute penalty.
8. A passing quiz score unlocks a limited reward rather than unrestricted entertainment access.
9. Focus mode should help block selected distracting applications such as social media, video, and games.
10. The application should work across web and Android development workflows.

## 3. Work completed to date

### 3.1 Product experience and navigation

The application has a branded Exam Focus interface with a responsive dashboard. The main experience includes Today, Study, Guide, Focus, Progress, timetable, quiz, reward, and penalty flows. On wider web screens, the interface uses a sidebar layout. On narrow web screens and mobile, it adapts to a native-style bottom-tab layout.

The dashboard communicates the study contract clearly: the student studies first, proves recall through a quiz, and earns focused screen time only after completing the required work.

### 3.2 Study catalog

The study catalog was expanded beyond the initial Physics examples. It now includes representative topics for the following pathways:

| Education pathway | Representative coverage |
|---|---|
| School | Mathematics, Science, English, Social Studies foundations |
| Intermediate | Mathematics, Physics, Chemistry, Biology, English and core examination topics |
| B.Tech | Engineering mechanics, DC circuits, materials and stress, programming, data structures and related fundamentals |
| Medical | Anatomy, physiology, cell biology, pathology-oriented foundations and medical study topics |
| Commerce | Accounting, economics, business studies and financial fundamentals |
| Humanities | History, civics, geography, psychology and related humanities topics |
| Computer science | Programming, algorithms, data structures and computer science fundamentals |
| Competitive examinations | General aptitude, reasoning, science, mathematics and examination-oriented revision topics |

The mobile Expo catalog and web catalog were kept consistent during the expansion. Cross-platform catalog consistency was also tested.

### 3.3 Study, quiz, reward, and penalty loop

The reward and penalty logic is implemented in shared application logic and is exercised by deterministic tests. The current behavior includes the following rules:

- A student must study a topic before taking the related quiz.
- A quiz can be configured with a passing score threshold.
- A result above the configured threshold produces reward minutes.
- The reward balance is limited and shown to the student.
- A student cannot use reward time while a penalty is active.
- Leaving a quiz before completion activates a 30-minute penalty.
- The quiz lock is cleared correctly after completion or abandonment.
- Web quiz state is persisted so a refresh does not silently remove the lock.
- Navigation and chatbot access are restricted while a quiz is in progress.

The default product target remains an 80% passing threshold with a configurable reward duration within the 15–30 minute range.

### 3.4 Timetable upload and AI parsing

The timetable flow accepts supported PDF and image formats. The uploaded file is converted into a server request and sent to the configured multimodal LLM. The model proposes examination information, including a date and subject, for the student to review and confirm.

A controlled timetable fixture was tested successfully. The parser returned a proposed date for a Physics examination, and the application presented the proposal for confirmation instead of silently changing the student’s plan.

The current parser is suitable for the MVP. Handling several examination dates from one timetable upload remains a planned enhancement.

### 3.5 AI study assistant

The assistant has been integrated into both the mobile tRPC flow and the web guide flow. It now uses the server-side Forge LLM rather than relying on mocked answer templates.

The implementation includes:

- Runtime model discovery through the provider model catalog.
- Selection of `gpt-5-mini` when it is available, with catalog-based selection logic.
- Server-side use of `BUILT_IN_FORGE_API_URL` and `BUILT_IN_FORGE_API_KEY`.
- A focused system prompt that asks for a direct explanation, one practical next step, and one recall question.
- Correct GPT-5 completion-token handling using `max_completion_tokens` at the provider request layer.
- Separate assistant behavior for topic tutoring and general study guidance.
- Explicit retryable error messages when the provider is unavailable.
- Removal of the old hard-coded template fallback from the active mobile and web response paths.

A live Physics request was tested successfully and returned a generated explanation about Newton’s second law and acceleration. Different questions are now processed by the LLM rather than being mapped to the same fixed text.

### 3.6 Focus mode and Android blocking

The app includes a focus plan where students select distraction categories and enable focus mode. The native Android implementation contains an optional Expo local module with an AccessibilityService bridge. The service can monitor foreground application changes and return the user to Exam Focus when a selected distraction package is opened during an active focus plan.

The native bridge includes:

- An Expo module configuration file.
- An Android Kotlin module.
- An Android AccessibilityService.
- An accessibility-service XML configuration.
- An Android manifest registration.
- A JavaScript bridge that safely becomes a no-op on web and in environments without the custom native module.
- A settings action that directs the user to Android Accessibility settings.

This is a real native enforcement path for custom Android builds. It is not fully available in standard Expo Go because Expo Go cannot load project-specific native modules. A development build or standalone Android build is required for actual device-level blocking.

The product must clearly explain that AccessibilityService access is sensitive device permission. The user should explicitly select the apps to block and be able to disable focus mode according to the product’s intended trust model.

### 3.7 Development and compatibility work

The project was upgraded from Expo SDK 54 to Expo SDK 57. The upgrade included compatible versions of Expo Router, React Native, Expo modules, React Navigation dependencies, Reanimated, Safe Area Context, SVG, and related packages.

The following compatibility problems were repaired:

- Expo Router native entry-point configuration.
- NativeWind and Tailwind module loading under the project’s module configuration.
- Metro and Vite development-server separation.
- Expo SDK 57 Pressable and navigation button typing.
- Material icon mapping types.
- Color scheme typing in the theme provider.
- Stale Expo configuration properties rejected by SDK 57 type definitions.
- Startup performance by using Expo offline mode and skipping nonessential dependency checks during development.
- Vite host restrictions for the managed preview hostname family.

The development scripts were subsequently adjusted for npm and Windows CMD. The default development command uses fixed ports rather than Unix-only shell parameter expansion.
