# tool-svg-utils

SVG 관련 유틸 2종을 탭 구조로 묶은 **단일 HTML** 웹 도구입니다.
모든 처리는 브라우저 안에서만 이루어지며 **어떤 파일도 서버로 전송되지 않습니다.**

🔗 **라이브 데모:** <https://hanariago.github.io/tool-svg-utils/>

---

## 기능

### 탭 1 — SVG 뷰어 / 미리보기
- `.svg` 파일 업로드(드래그&드롭) 또는 코드 직접 붙여넣기
- 실시간 렌더링 미리보기 (투명 배경 체커보드 표시)
- **PNG / JPG 다운로드** — 해상도 배율 선택(1× ~ 4×) 또는 가로 px 직접 입력
- SVG 원본 저장 / 코드 보기·숨기기 토글

### 탭 2 — 이미지 → SVG 변환
- PNG · JPG · WebP · GIF 업로드
- 두 가지 변환 방식 제공 (각 방식의 특성/권장 용도 안내 포함):

| 방식 | 원리 | 권장 용도 |
|------|------|-----------|
| **벡터 트레이싱** | 윤곽선을 실제 `<path>` 벡터로 추적. 무한 확대 가능 | 로고, 아이콘, 단순 일러스트, 흑백 스캔 |
| **SVG Embed** | 원본 이미지를 base64로 SVG 안에 그대로 삽입. 화질 100% 보존 | 컬러 사진, 복잡한 그래픽 |

- 트레이싱: 흑백/컬러 모드, 색상 수, 디테일 단계 선택
- 변환 결과 미리보기 → SVG 다운로드 (PNG 저장 / 코드 보기 보조)

> 모바일 뷰포트 대응(반응형 레이아웃).

---

## 구현 메모 (라이브러리 선택 근거)

요청된 리서치 결과를 반영해 다음과 같이 결정했습니다.

- **벡터 트레이싱은 [ImageTracer.js](https://github.com/jankovicsandras/imagetracerjs)를 채택** (Potrace 대신)
  - **라이선스:** ImageTracer.js는 **Unlicense(퍼블릭 도메인)** 라 repo 라이선스를 자유롭게 가져갈 수 있음.
    반면 브라우저용 Potrace 빌드(`esm-potrace-wasm`)는 원본 Potrace를 따라 **GPL-2.0** 이라, 채택 시 이 repo 전체가 GPL 전염됨.
  - **구동:** ImageTracer는 순수 JS라 CDN 한 줄로 끝나 "단일 HTML" 원칙에 부합. Potrace WASM은 별도 번들/빌드 스텝 필요.
  - **기능:** ImageTracer는 흑백·컬러 트레이싱을 모두 지원.
- **SVG → PNG/JPG 변환은 `data:` URI 방식 사용**
  - `blob:` / `http:` URL로 SVG를 canvas에 그리면 Safari·모바일 Chrome에서 canvas가 *tainted* 되어 `toBlob()`/`toDataURL()`이 실패함.
  - SVG를 `encodeURIComponent`로 직렬화한 **data URI**로 그리면 모든 브라우저에서 taint 없이 변환됨 (foreignObject 미포함 사용자 SVG 기준).
  - JPG는 알파 채널이 없으므로 변환 전 흰색 배경을 먼저 채움.

## 보안 설계

- **SVG 미리보기는 `<img>` + data URI로 렌더링** — 붙여넣은 SVG를 `innerHTML`로 넣지 않음.
  이미지 컨텍스트에서는 SVG 내부의 `<script>` / `onload` 등이 **실행되지 않으므로**, 악의적으로 작성된
  SVG를 미리보기해도 스크립트가 실행되지 않음(self-XSS 차단). GitHub Pages처럼 여러 프로젝트가 같은
  origin을 공유하는 환경에서 특히 중요.
- **CDN 스크립트에 Subresource Integrity(SRI) 적용** — ImageTracer.js를 `integrity="sha384-…"`로 고정.
  CDN이 변조되거나 중간자 공격이 있어도 해시 불일치 시 브라우저가 로드를 거부함.
  해당 해시는 npm 레지스트리 정식 배포본(v1.2.6)과 바이트 단위로 동일함을 확인함.
- 네트워크 통신은 위 CDN 스크립트 1건뿐이며, **업로드한 이미지·SVG 데이터는 어떤 경우에도 외부로 전송되지 않음.**

---

## 로컬 실행

빌드 과정이 없습니다. `index.html`을 브라우저로 열기만 하면 됩니다.

> 단, 일부 브라우저는 `file://` 보안 정책으로 CDN 스크립트를 막을 수 있어, 간단한 로컬 서버 사용을 권장합니다.

```bash
python -m http.server 8000
# http://localhost:8000 접속
```

---

## GitHub Pages 배포

1. 이 repo를 **Public**으로 GitHub에 push
2. **Settings → Pages → Build and deployment**
   - Source: `Deploy from a branch`
   - Branch: `main` / 폴더: `/ (root)`
3. 잠시 후 `https://<username>.github.io/tool-svg-utils/` 에서 접속

---

## 라이선스

- **이 프로젝트의 소스 코드:** [MIT License](LICENSE)
- **사용 라이브러리:**
  - [ImageTracer.js](https://github.com/jankovicsandras/imagetracerjs) — Unlicense (퍼블릭 도메인). CDN(SRI 고정): `cdn.jsdelivr.net/npm/imagetracerjs@1.2.6/imagetracer_v1.2.6.js`

MIT와 Unlicense는 모두 상업적 사용을 포함해 자유롭게 사용·수정·배포할 수 있습니다.
