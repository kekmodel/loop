# loop

Codex에서 반복 작업과 후속 확인 일정을 만들거나 수정하는 스킬입니다. 독립 실행과 현재 대화의 맥락을 이어가는 실행을 구분하고, 실행 주기·보고 조건·종료 조건·입력이 필요한 상황을 정리합니다.

## 설치

Git과 GitHub CLI를 준비하고, 이 비공개 저장소에 접근할 수 있는 계정으로 로그인합니다.

```bash
gh auth login
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
gh repo clone kekmodel/loop "${CODEX_HOME:-$HOME/.codex}/skills/loop"
```

대상 폴더가 이미 있다면 기존 파일을 백업한 뒤 설치하세요.

## 사용

새 메시지에서 `$loop`와 요청을 함께 입력합니다.

- `$loop 매주 월요일 오전 9시에 이 프로젝트의 지난주 변경 사항을 새 작업으로 정리해줘.`
- `$loop 이 배포를 10분마다 확인하고 완료되거나 실패하면 알려줘.`

실제 일정 생성에는 Codex 환경의 예약 도구가 필요합니다. 스킬 파일만 설치해도 예약 기능 자체가 추가되는 것은 아닙니다. 지원 여부와 작업 방식은 실행 환경의 도구 지침을 따릅니다.

## 업데이트

위 명령으로 설치한 복제본은 다음 명령으로 업데이트합니다.

```bash
git -C "${CODEX_HOME:-$HOME/.codex}/skills/loop" pull --ff-only
```

## 구성

- `SKILL.md`: 작업 유형 선택, 일정·보고·종료 조건, 예약 도구 사용 지침
- `agents/openai.yaml`: 표시 이름과 짧은 설명
