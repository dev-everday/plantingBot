# 잔디심기 봇 README

GitHub 잔디 심기를 자동화하는 **잔디심기 봇**(이하 PlantingBot) 코드입니다.

## 📌 프로젝트 개요
GitHub 커밋 히스토리를 관리하기 어려워 고민이 많으신가요?
매일 커밋해서 나만의 잔디 운동장을 만들고 싶지만 바쁜 일정에 놓치기도 하죠.
**GitHub Actions**를 설정하면 매일 자동으로 커밋을 생성해 잔디를 심을 수 있습니다.

---

## 🛠️ 준비 사항
### 🔧 GitHub Repository 설정 변경
1. **Repository Settings**로 이동합니다.
2. **Actions > General**로 이동합니다.
3. **Workflow permissions**을 **Read and write permissions**로 설정합니다.

### 🔒 암호화 변수 설정 (선택 사항)
1. **Settings > Secrets and variables > Actions**로 이동합니다.
2. `New repository secret`을 클릭하고, 변수 이름을 `USER_EMAIL`로 지정합니다.

---

## 🚀 GitHub Actions Workflow 설정
### 📂 새 Workflow 생성
1. **Actions** 탭으로 이동합니다.
2. **New workflow** 버튼을 클릭합니다.
3. **Simple workflow** 선택 후 `.github/workflows/blank.yml` 파일을 생성합니다.
4. 파일 이름을 `planting-bot.yml`로 지정합니다.

### 💻 예시 코드
아래는 매일 오후 9시 00분 기준으로 커밋 여부를 확인하고, 커밋하지 않았다면 자동으로 커밋하는 GitHub Actions 스크립트입니다.

```yaml
name: daily planting bot

on:
  workflow_dispatch:
  schedule:
    - cron: '0 12 * * *'

jobs:
  auto-commit:
    runs-on: ubuntu-latest
    steps:
      - name: 저장소 체크아웃
        uses: actions/checkout@v3
        with:
         fetch-depth: 0

      - name: 오늘 커밋 여부 확인
        id: check_commit
        run: |
          DATE=$(TZ=Asia/Seoul date '+%Y-%m-%d')
          echo "오늘 날짜: $DATE"
          if git log --since="$DATE 00:00 +0900" --author="$GITHUB_ACTOR" --oneline | grep .; then
            echo "오늘 이미 커밋이 있습니다. 워크플로를 종료합니다."
            echo "should_run=false" >> $GITHUB_OUTPUT
          else
            echo "오늘 커밋이 없습니다. 자동 커밋을 진행합니다."
            echo "should_run=true" >> $GITHUB_OUTPUT
          fi

      - name: 파일 업데이트
        if: steps.check_commit.outputs.should_run == 'true'
        run: |
          DATE=$(TZ=Asia/Seoul date '+%Y%m%d')
          echo "DATE=${DATE}" >> $GITHUB_ENV  # DATE 변수를 환경 변수로 설정
          echo "🌿오늘은 bot이 잔디 심어요🌿" > "[${DATE}]"

      - name: 변경 사항 커밋
        if: steps.check_commit.outputs.should_run == 'true'
        run: |
          echo "사용 중인 DATE 변수: ${DATE}"
          git config --global user.name "$GITHUB_ACTOR"
          git config --global user.email "${{ secrets.USER_EMAIL }}"
          ls -l  # 현재 디렉토리의 파일 목록 확인
          git add "[${DATE}]"
          git commit -m "자동 커밋: ${DATE}"

      - name: 자격 증명 설정
        if: steps.check_commit.outputs.should_run == 'true'
        run: |
          git config --global user.name "$GITHUB_ACTOR"
          git config --global user.email "${{ secrets.USER_EMAIL }}"
          git remote set-url origin https://x-access-token:${{ secrets.GITHUB_TOKEN }}@github.com/${{ github.repository }}

      - name: 변경 사항 푸시
        if: steps.check_commit.outputs.should_run == 'true'
        run: |
          git push
```

---

## ⚠️ 주의 사항
- **커밋 메시지**: 원하는 커밋 메시지를 자유롭게 수정하세요.
- **GitHub Actions 활성화**: Actions 탭에서 해당 Workflow가 활성화되어 있는지 확인하세요.

---

## 📬 문의
- **이슈 발생 시**: GitHub Issues 또는 댓글로 남겨주시면 확인 후 답변드리겠습니다.

즐거운 잔디심기 되세요! 🌱

