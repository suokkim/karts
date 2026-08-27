# 새 학기 업로드 절차

학기가 끝날 때마다 이 문서대로 하면 된다. 처음부터 끝까지 30분이면 끝난다.
크기·포맷 규격은 [MEDIA_POLICY.md](MEDIA_POLICY.md) 가 정본이다.

작업 폴더는 `~/Documents/karts-studioF/`. 그 안의 **`_web/` 폴더 자체가
GitHub 저장소**(`suokkim/karts`)이고, 그게 그대로 https://suokkim.github.io/karts/ 로 나간다.

---

## 0. 자료 모으기 — 두 갈래

학기 자료가 **노션에 있으면 A**, **파일로 받았으면 B**. 섞여 있으면 A 먼저 하고 B 를 덧붙인다.

### A. 노션에 있을 때

1. 노션에서 **무대미술 - 스튜디오-F** 페이지 열기
2. 우측 상단 `⋯` → **내보내기** → 형식 **Markdown & CSV**, **하위 페이지 포함** 켜기
3. 받은 zip 을 **`ditto` 로 푼다** (⚠️ `unzip` 은 한글 파일명을 깨뜨린다):

```bash
cd ~/Downloads
ditto -x -k <받은파일>.zip out          # 안에 zip 이 또 들어 있으면 한 번 더
```

4. 압축 안의 학기 폴더를 `~/Documents/karts-studioF/<연도>-<학기>/` 로 옮기고,
   파일명 끝의 노션 해시를 뗀다 — `_organize.py` 가 하던 일이다
   (최초 이관 때 쓴 스크립트라 그대로는 안 돌아간다. 경로만 맞춰 쓰거나 손으로 옮겨도 된다)

### B. 파일로 받았을 때 (2026-1 이 이 경우)

학생 이름 폴더를 그대로 학기 폴더 밑에 둔다. 그게 곧 갤러리의 묶음 단위가 된다.

```bash
cd ~/Documents/karts-studioF
mkdir -p 2026-2/배연재
mv ~/Downloads/배연재/* 2026-2/배연재/
find 2026-2 -name '.DS_Store' -delete
```

---

## 1. 웹용으로 변환

원본은 학기 폴더에 그대로 두고, `_web/` 에 경량 사본만 만든다.
**새로 넣은 학기만** 돌리면 된다 (전체 재변환은 몇 분 걸린다).

```bash
cd ~/Documents/karts-studioF
_venv/bin/python -c "
from pathlib import Path; import _optimize as O
[O.handle(f.resolve()) for f in sorted(Path('2026-2').rglob('*')) if f.is_file()]"
```

이미지는 JPEG, 영상은 H.264, PDF 는 페이지마다 JPEG 로 바뀐다.
`_venv/bin/python` 을 쓰는 이유는 PDF 처리용 pymupdf 가 거기에만 있어서다.

## 2. 갤러리 다시 만들기

```bash
python3 _site.py
```

새 학기 카드가 맨 앞에 생기고, 학기 페이지가 만들어진다. 학생 이름이 명단에 없으면
`_site.py` 의 `NAMES` 에 추가해야 한 사람으로 묶인다 (아래 4절).

## 3. 검사 — **`git add` 를 먼저 하고 검사한다**

```bash
cd _web && git add -A && cd ..
python3 _checkrefs.py        # 링크가 git 에 있는 파일을 가리키는지
node _testviewer.mjs         # 누른 썸네일과 열리는 그림이 같은지
```

`_checkrefs.py` 는 로컬 파일이 아니라 **`git ls-files`** 와 대조한다. add 를 안 하면
새 파일이 전부 "없음" 으로 잡히니 순서를 지킬 것. 둘 다 통과해야 올린다.

## 4. 올리기

```bash
cd _web
git commit -m "2026-2 추가"
git push
```

1~2분 뒤 https://suokkim.github.io/karts/ 에 반영된다. 확인:

```bash
gh api repos/suokkim/karts/pages/builds/latest --jq .status   # built 면 완료
```

브라우저에 옛 화면이 남으면 강력 새로고침(⌘⇧R).

---

## 자주 걸리는 것

| 증상 | 원인·해결 |
|---|---|
| 한글 파일명이 `ۦ��ة` 처럼 깨짐 | `unzip` 을 썼다. `ditto -x -k` 로 다시 풀 것 |
| 로컬에선 보이는데 사이트에서 이미지 404 | 링크가 NFD(자모 분리)다. `_site.py`/`_optimize.py` 는 NFC 로 적는다. `_checkrefs.py` 가 잡아준다 |
| `_checkrefs.py` 가 새 파일을 전부 없다고 함 | `git add -A` 를 먼저 안 했다 |
| 한 학생 작업이 갤러리에서 여러 묶음으로 흩어짐 | `_site.py` 의 `NAMES` 에 그 이름을 추가 |
| 제목 없는 폴더가 그대로 보임 | `_site.py` 의 `GROUP_ALIAS` 에 `(학기, 폴더): 표시이름` 등록 |
| 학기 표지가 하얀 문서 이미지 | 자동으로 걸러진다. 그래도 이상하면 `looks_like_text()` 의 문턱(흰 픽셀 45%, 중간톤 20%) 조정 |
| PDF 변환에서 `No module named pymupdf` | `python3` 대신 `_venv/bin/python` 을 쓸 것 |

## 건드리면 안 되는 것

- **`_web/.git` 삭제 금지.** 재변환할 때 `rm -rf _web` 하면 저장소가 통째로 날아간다.
  `_optimize.py` 는 덮어쓰기라 지울 필요가 없다
- 원본(학기 폴더)은 지우지 않는다. 규격을 바꿔 다시 뽑을 때 필요하다 (2.4GB, 로컬 전용)
- 확대 뷰어를 `<dialog>` 로 되돌리지 말 것 — iOS 에서 문제가 있어 지금 구조로 왔다

## 지금 상태 (2026-08-27)

- 15개 학기 (2018-2 ~ 2026-1, 2024-2 는 원본에 없음), 그룹 71개, 작품 836점
- 원본 2,432MB / 공개본 **159MB** (6.5%)
- 스크립트: `_optimize.py`(변환) · `_site.py`(갤러리) · `_checkrefs.py`·`_testviewer.mjs`(검사)
  · `_organize.py`·`_checklinks.py`(최초 이관용, 기록)
