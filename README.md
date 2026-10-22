# NDL · Manual Simulator

NDL은 연구실 장비의 조작 순서를 사진과 화면을 보며 연습하는 Windows용 오프라인 앱입니다. 장비와 절차를 선택하고, 버튼·밸브·확인 항목을 따라 진행합니다. 필요한 부분은 확대해서 보고, 중간에 저장한 연습은 나중에 이어갈 수 있습니다.

현재 문서 기준 버전은 **v1.18.0**입니다. 실행 파일명은 `Lab-Offline-Trainer.exe`입니다.

## 지원 장비

| 장비 | 학습 내용 |
| --- | --- |
| VPS 진공 챔버 | 시료 투입, 배기, 측정 준비, 회수. 고진공 복귀와 전체 종료 두 경로, 터보 재기동 분기 |
| AFM · XEP | 시료·팁 장착, 정렬, Approach, 스캔, 정리와 시료 회수 39단계 |
| E-beam 증착기 | 공동실험실 매뉴얼 78단계와 제작 공정 매뉴얼 50단계 |
| Raman · RAON-Spec / RAON-Vu | RAON-Spec 절차 36단계와 RAON-Vu 화면 탐색 |
| EBL · TESCAN MIRA3 | Elphy 문서 기반 절차 45단계와 DrawBeam 화면 연습 40단계 |

장비별로 사진 조작, 화면 탐색, 문서 확인을 함께 사용합니다. 박리·전사, Spin coating, Baking, Lift-off는 오프라인 매뉴얼로 볼 수 있습니다.

## 주요 기능

- 사진의 버튼·스위치·밸브를 눌러 단계 진행
- VPS 도트 패널, AFM 단계별 사진과 확대 이미지
- 사진 확대·이동과 조작 위치 강조
- 조작 순서와 상태에 따른 진행 확인, 오류 기록
- 안내 연습·자율 연습·평가 모드
- 사용자별 PIN 로그인, 진행 저장·재개, 새 연습 시작
- 결과 요약과 JSON 내보내기
- 포함된 원문 PDF의 오프라인 열람

## Windows에서 실행하기

1. Windows 배포 ZIP을 쓰기 가능한 새 폴더에 모두 풉니다.
2. `Lab-Offline-Trainer.exe` 또는 `Start.cmd`를 실행합니다. 실행 파일만 따로 옮기지 마세요.
3. 처음 실행하면 이름과 숫자 PIN으로 프로필을 만듭니다.
4. 장비 → 절차 → 연습 순서로 선택합니다. 현재 단계의 조작을 마치고 **다음**을 누릅니다.
5. 중간에 멈출 때는 **저장 후 나가기**, 다시 시작할 때는 **학습 재개**를 사용합니다.

배포본 실행에는 Python이나 Node.js 설치가 필요하지 않습니다. 이미지와 PDF도 배포 폴더에 포함됩니다.

## 기록 저장과 백업

기본 저장 위치는 실행 파일 옆의 `Lab-UserData/`입니다.

- `profiles.json`: 프로필과 PIN 인증 정보
- `records/`: 사용자별·장비별 현재 연습 기록
- `history/`: 새 연습을 시작할 때 보관한 이전 결과
- `content/`: 앱에서 저장한 사용자 수정 콘텐츠
- `runtime/`: 앱 실행 환경 데이터

앱을 종료한 뒤 **`Lab-UserData` 폴더 전체를 복사**하면 백업할 수 있습니다. 새 버전은 별도 폴더에 풀고, 기록을 이어 쓰려면 백업 복사본을 새 실행 파일 옆에 둡니다. 기존 배포본과 원본 기록은 남겨 두세요. 절차 버전이 맞지 않는 기록은 재개가 제한될 수 있습니다.

저장 파일을 갱신할 때 기존 파일의 `.bak` 사본도 만듭니다. 결과 화면의 JSON 내보내기는 개별 결과 공유용이며 전체 기록 백업과는 별개입니다. `Lab-UserData/`는 Git에 올리지 않습니다.

## 개발 실행과 Windows 패키징

아래 명령은 `app/`, `tools/`, `docs/`가 있는 소스 루트에서 **PowerShell**로 실행합니다. Node.js와 npm이 필요합니다.

현재 `app/package.json`에는 `main: main.cjs`가 지정되어 있으며, npm `start`·`build` 스크립트와 의존성 목록은 없습니다. Electron을 별도 도구 폴더에 설치해 실행합니다. v1.18.0 배포에 사용한 Electron 버전은 **44.7.0**입니다.

```powershell
# 개발용 Electron 설치
npm install --prefix .dev-tools --no-save --package-lock=false electron@44.7.0

# 개발 기록은 배포본 기록과 분리
$env:VPS_DATA_DIR = Join-Path (Get-Location).Path '.dev-data'
$env:ELECTRON_RUN_AS_NODE = ''
& .\.dev-tools\node_modules\electron\dist\electron.exe .\app
```

Windows 실행 폴더를 만들려면 다음 명령을 실행합니다. `ELECTRON_DIST`를 지정하면 패키징 스크립트의 로컬 기본 경로 대신 설치한 Electron을 사용합니다.

```powershell
$env:ELECTRON_DIST = (Resolve-Path '.\.dev-tools\node_modules\electron\dist').Path
node .\tools\package.cjs
```

결과는 `release/Lab-Offline-Trainer/`에 생성됩니다. 이 폴더 전체가 배포 단위입니다. 패키징 후 배포본을 확인할 때는 개발용 저장 경로 설정을 해제합니다.

```powershell
Remove-Item Env:VPS_DATA_DIR -ErrorAction SilentlyContinue
& .\release\Lab-Offline-Trainer\Lab-Offline-Trainer.exe
```

자동 테스트는 다음 명령으로 실행합니다. 소스에 포함된 `tests/fixtures/`와 `evidence/`의 비교 자료도 필요합니다.

```powershell
node .\tools\run-unit.cjs
```

결과는 `evidence/unit-test-report.txt`에 저장됩니다. Electron 화면 테스트 중에는 로컬 경로를 사용하는 스크립트가 있으므로 별도 환경 설정이 필요합니다.

## 소스 구조

```text
app/
  package.json          앱 정보와 시작 파일
  main.cjs              창, 로컬 저장, IPC 처리
  preload.cjs           화면과 앱 사이의 API
  renderer.js           장비 선택·학습·결과 화면
  *content.json         장비별 단계와 안내, 사진 매핑
  *engine.js            장비 상태와 단계 전이
  assets/               사진·도트 이미지·아이콘
  manuals/              원문 PDF
  pages/                매뉴얼 페이지 자료
  vendor/               포함된 라이브러리
  viewer.*              PDF 열람 화면
  *.css                 공통·장비별 화면 스타일
docs/                   변경 내역과 검증 문서
tests/                  자동 테스트와 비교 자료
tools/                  패키징·검증 도구
evidence/               테스트 결과와 비교용 목록
START-HERE.md           배포본 시작 안내
```

## 저장소와 배포 파일

Git 저장소에는 소스와 필요한 문서를 관리하고, 생성된 `release/`, `.dev-tools/`, `.dev-data/`, `Lab-UserData/`는 제외합니다. 100 MB가 넘는 실행 파일이나 배포 ZIP은 소스 커밋에 넣지 않습니다. Windows 배포 ZIP은 GitHub Releases 등의 별도 배포 공간에 게시하는 구조로 관리합니다.
