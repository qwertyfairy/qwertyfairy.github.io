# 하루의 기록 — 운영 규칙

Jekyll + **Chirpy** 테마 기반 개인 블로그. GitHub Pages에 **GitHub Actions**로 배포.
이 문서는 사람용 운영 명세이자 Claude Code 작업 지침이다. (`CLAUDE.md`가 이 파일을 가리킴)

> **문서 갱신 규칙**: 이 블로그 관련 수정·정책이 **확정될 때마다 이 문서를 갱신**한다.
> 새 규칙이 정해지면 아래 해당 섹션에 즉시 반영할 것.

## 새 글 쓰기

1. `_posts/`에 `YYYY-MM-DD-제목.md` 형식으로 파일 생성 (한글 제목·공백 허용)
2. front matter:
   ```yaml
   ---
   title: "글 제목"
   categories: [상위, 하위]
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
- **표기 이름**: SHISA (작성자·저작권, `_config.yml`의 `social.name`). 푸터 이름 링크는 GitHub (`social.links` 첫 항목)
- **X(트위터) 미사용**: 링크·meta 어디에도 넣지 말 것
- **아바타**: `/public/img/avatar.jpeg` (테두리 1px는 커스텀 CSS)
- **파비콘**: 주황 고양이 아이콘 (`assets/img/favicons/`, 원본: iconify solar:cat-bold #E8833A)
- **태그 미사용**: 태그 탭 제거 (`_tabs/tags.md` 삭제). 포스트에 `tags` front matter 쓰지 말 것
- **한글 폰트**: Pretendard. `_includes/metadata-hook.html`에서 jsDelivr CDN 로드, `assets/css/jekyll-theme-chirpy.scss`에서 `--bs-font-sans-serif` override (코드블록 monospace는 유지)
- **콜아웃(prompt)**: `prompt-info` 사용. 색은 머스터드 `#dda233`, ⓘ 아이콘 제거, 콜아웃 안 리스트는 자체 여백 0(콜아웃 패딩이 간격 담당). 모두 `assets/css/jekyll-theme-chirpy.scss`의 전역 CSS라 새 글에 자동 적용됨 — 포스트마다 손댈 필요 없음
- **Q&A 박스**: 질문형 콘텐츠(Q. ~)는 `<div class="box-qa" markdown="1"> … </div>`로 감싼다. 세이지 그린 `#9ab87a` + 왼쪽 강조선, 코드블록 포함 가능. 질문→답→코드→마무리를 한 박스에 담아 흐름이 끊기지 않게 한다. 박스 안 소제목은 헤딩(`####`) 대신 **볼드** 사용(TOC에 안 잡히게)
- 사이트 제목/작성자/이메일 등은 `_config.yml`의 `title`, `social`에서 관리

## 작업 방식: 미리보기 먼저, 배포는 승인 후

수정은 이 저장소(main)에 바로 커밋하지 않는다. **미리보기 샌드박스 `~/AndroidStudioProjects/chirpy-preview`** 에서 작업해 `localhost:4321`로 사용자에게 보여주고, **최종 승인을 받은 뒤에만** 이 저장소에 반영·push한다.

- push 전 필수 검증: `JEKYLL_ENV=production` 빌드 + htmlproofer 통과 (배포 워크플로가 같은 검사를 하므로 실패하면 배포가 안 됨)
- 미리보기 저장소가 없으면: 이 저장소를 복제해 `_config_dev.yml`(아래) 방식으로 띄우면 동일

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

## 주의: `url`/`baseurl` 설정

- 유저 사이트(`qwertyfairy.github.io`)라 **`baseurl`은 반드시 `""`**, `url`은 `"https://qwertyfairy.github.io"`.
- `baseurl`에 도메인을 넣으면 모든 링크가 이중 프리픽스(`/https://.../...`)로 깨지고, 배포 워크플로의 htmlproofer 테스트가 실패한다.
- 포스트 이미지는 `public/img/`에 두고 `/public/img/파일명`으로 참조. 이미지 파일이 없으면 htmlproofer가 빌드를 실패시킨다.

## 주의: 전역 gitignore

이 맥의 `~/.gitignore_global`에 `*.yml`이 있어 **.yml 파일이 기본적으로 커밋에서 누락**된다.
저장소 `.gitignore`의 `!*.yml`로 무력화해둠 — **이 줄을 지우면 워크플로·데이터 파일이 다시 사라지니 유지할 것.**
