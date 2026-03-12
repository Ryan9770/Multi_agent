# AutoGen V2: Enterprise Multi-Agent Game Development Pipeline

이 프로젝트는 Microsoft AutoGen 웹 게임 개발 파이프라인을 엔터프라이즈급으로 고도화한 자동화 프레임워크입니다. 다중 AI 에이전트(기획, 아키텍처, 엔지니어, 리뷰어, QA, 문서화)가 파트별로 협업하여 유저의 요구사항에 따라 웹 게임을 "원클릭"으로 기획부터 테스트까지 자동 생성합니다.

## 🚀 주요 특징 (Key Features)

1. **FSM (유한 상태 기계) 기반 통제력 확보**
   - 기존 AutoGen의 무작위 '자유 토론(Open Chat)' 방식이 아닌, `allowed_or_disallowed_speaker_transitions`를 사용하여 에이전트 간의 발언권(Speaker) 순서를 엄격하게 통제합니다.
   - **워크플로우:** `방장(Admin)` ➔ `총괄 디렉터` ➔ `기획` ➔ `아트` ➔ `아키텍트` ➔ `개발` ➔ (`개발` ↔ `코드 리뷰` ↔ `QA` 재작성 루프) ➔ `문서화` ➔ `종료`

2. **완벽한 코드 모듈화 (Vanilla JS/CSS/HTML 분리)**
   - LLM이 단일 파일에 모든 코드를 뭉뚱그려 내뱉는 것을 방지합니다.
   - `System_Architect` 에이전트가 파일 시스템 구조를 설계하고, `Engineer`가 Python 스크립트를 출력하여 `.html` 1장, 다수의 `.js` 모듈, `.css` 파일들을 `output/` 디렉토리에 물리적으로 분할하여 저장(I/O)합니다.

3. **고도화된 프롬프트 엔지니어링 및 AI 게으름 방지 로직**
   - 개발을 담당하는 AI가 `// TODO: Implement`와 같은 빈 껍데기(Skeleton) 코드만 내뱉고 턴을 넘기는 "AI 파업/게으름(Laziness)" 현상을 차단하기 위해 **강력한 페널티 프롬프트**가 적용되어 수백 줄 이상의 실제 작동하는 코드를 작성하도록 강제합니다.

4. **자체 코드 리뷰 및 QA(품질 보증) 무한 루프**
   - **`Code_Reviewer`**: 개발된 로직과 변수 스코프 충돌을 깐깐하게 점검하고, 버그 발견 시 개발자에게 재작성 및 수정을 지시(반려)합니다.
   - **`QA`**: Playwright 엔진을 통한 `test_game` 도구를 사용하여 헤드리스 브라우저 환경에서 생성된 게임(index.html)을 실제 로드하여 JS Syntax 에러나 런타임 크래시를 검출해 개발자에게 보고합니다.

5. **API Rate Limit (호출 제한) 자동 방어**
   - 구글 Gemini Free Tier의 분당 요청 제한(15 RPM) 등 무료 API의 한계를 우회하기 위해 `custom_speaker_selection` 함수 내부에 4.5초의 인위적 딜레이(Throttle)를 적용하여 안정적인 장기 파이프라인 구동이 가능하게 설계되었습니다.

## 🛠 시스템 요구 사항 (Prerequisites)

- Python 3.10 이상
- `pyautogen` (AutoGen 코어)
- `playwright` (QA 테스트용 헤드리스 웹 브라우저)
- `google-generativeai` 또는 `google-genai` (Google LLM 연동)

## 📦 설치 및 실행 (Installation & Usage)

1. **가상환경 활성화**
   ```bash
   # Windows PowerShell 기준
   .\vevn\Scripts\Activate.ps1
   ```

2. **종속성 설치 및 Playwright 브라우저 셋업**
   ```bash
   pip install pyautogen playwright google-genai
   playwright install
   ```

3. **API 키 설정**
   - `discussion_autogen.py` 파일을 열고 `llm_config` (및 `llm_config_coder`) 딕셔너리 내의 `api_key` 필드에 본인의 Google Gemini API 키를 입력하세요.
   - (보안을 위해 환경변수 `os.environ.get("GEMINI_API_KEY")`로 대체하는 것을 권장합니다.)

4. **파이프라인 실행**
   ```bash
   python discussion_autogen.py
   ```
   - 스크립트를 실행하면 기존 찌꺼기 `output/` 폴더가 초기화되고, 에이전트들의 회의가 시작됩니다.
   - 약 10~15분 후 (Rate Limit 딜레이 포함), QA 및 문서화가 끝난 완벽한 [2D 샌드박스 웹 게임]이 물리적인 폴더(`output/`)에 생성됩니다.

## 📁 디렉토리 구조 (Generated Output Structure)

실행이 성공적으로 완료되면 에이전트들이 자동으로 구축하는 `output` 폴더의 지형도입니다:

```text
output/
├── index.html          # 게임 메인 진입점 (단일 HTML)
├── css/
│   ├── reset.css
│   ├── style.css       # 전역 스타일 및 레이아웃
│   └── game.css        # 게임 내부 UI 스타일
├── js/
│   ├── main.js         # 실행 엔트리포인트 및 DOMContentLoaded
│   ├── constants.js    # 글로벌 상수 (TILE_SIZE, MAP_SIZE 등)
│   ├── game_state.js   # 상태 관리
│   ├── world_generator.js # 무한맵 및 청크, 바이옴 생성 로직
│   ├── player.js       # 플레이어 물리 조작(WASD), 중력 및 충돌
│   ├── renderer.js     # Canvas 2D 렌더링 엔진 
│   ├── inventory.js    # 자원 수집 및 인벤토리 로직
│   ├── crafting.js     # 제작/조합 대화상자 로직 
│   ├── input_handler.js # 키보드/마우스 입력 및 채굴 상호작용
│   └── utils.js        # 수학 연산 헬퍼 
└── assets/
    └── images/
        ├── blocks/     # 흙, 풀, 돌 등 블록 텍스처 (urllib 자동 다운로드)
        └── ui/         # UI 관련 에셋
```

## Tip
그럴듯한 게임을 만들고 싶다면 요구사항을 최대한 구체적으로 작성해야합니다.
또한, 무료 에셋 등을 추가하여 활용을 유도하는 것이 좋습니다.