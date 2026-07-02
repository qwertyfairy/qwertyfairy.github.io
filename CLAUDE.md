# 하루의 기록 — 운영 규칙

Jekyll + **Chirpy** 테마 기반 개인 블로그. GitHub Pages에 **GitHub Actions**로 배포.
이 문서는 사람용 운영 명세이자 Claude Code 작업 지침이다.

## 새 글 쓰기

1. `_posts/`에 `YYYY-MM-DD-제목.md` 형식으로 파일 생성 (한글 제목·공백 허용)
2. front matter:
   ```yaml
   ---
   title: "글 제목"
   categories: [상위, 하위]
   tags: [태그1, 태그2]   # 선택
   ---
   ```
3. 본문은 그 아래 Markdown으로 작성

`title`을 안 적으면 파일명에서 자동 추출되지만, **명시하는 걸 권장**.

## 카테고리 규칙

**2단계 계층**: `[플랫폼/언어, 성격]`

- 상위 = 플랫폼/언어: `Java`, `Android`, (필요시 `iOS`, `Backend` 등 추가)
- 하위 = 글 성격: `학습`(공부·개념 정리) / `작업`(실무에서 한 작업)

예: `[Java, 학습]`, `[Android, 작업]`

## 확정된 디자인/기능 규칙

- **언어**: `lang: ko-KR` — 메뉴·UI 전부 한글. About 메뉴는 "소개"로 override (`_data/locales/ko-KR.yml`)
- **읽는 시간 미표시**: `_includes/read-time.html`을 빈 파일로 override (지우지 말 것)
- **공유하기**: 소셜 플랫폼 없음, 링크 복사만 (`_data/share.yml`의 `platforms` 비움)
- **사이드바 연락처**: GitHub + 이메일만 (`_data/contact.yml`)
- 사이트 제목/작성자/이메일 등은 `_config.yml`의 `title`, `social`에서 관리

## 로컬 미리보기

Ruby 3.x 필요 (Chirpy는 `~> 3.1`, Ruby 4는 안 됨). 이 맥엔 `ruby@3.3` 설치돼 있음.

```bash
export PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH"
bundle install                 # 최초 1회
# 로컬은 url을 비워야 경로가 안 꼬임
printf 'url: ""\nbaseurl: ""\n' > _config_dev.yml   # 이미 있으면 생략, 커밋하지 말 것
bundle exec jekyll serve --config _config.yml,_config_dev.yml
# http://127.0.0.1:4321/
```

## 배포

`main` 브랜치에 push → `.github/workflows/pages-deploy.yml`이 자동 빌드·배포.
(GitHub 저장소 Settings → Pages → Source가 **"GitHub Actions"** 여야 함.)

## 주의: 전역 gitignore

이 맥의 `~/.gitignore_global`에 `*.yml`이 있어 **.yml 파일이 기본적으로 커밋에서 누락**된다.
저장소 `.gitignore`의 `!*.yml`로 무력화해둠 — **이 줄을 지우면 워크플로·데이터 파일이 다시 사라지니 유지할 것.**
