# AXIS · 힉스필드 4컷 프롬프트킷 (이음새 없는 체인)

> 목표: 분해 → 내부 줌 → 운전석 → 엔진/배터리 4컷을 **한 번의 카메라 무빙처럼** 연결.
> 모델: **Seedance 2.0** (start_image / end_image 지원 → 체인 연결 가능).

---

## ⛓️ 연결 규칙 (제일 중요)

1. **Shot 1**의 시작 이미지 = AXIS 히어로 정지컷 (예: `car3/f120.jpg` 또는 깨끗한 풀카 스튜디오 컷).
2. 생성 후 **마지막 프레임을 캡처** → 그게 **Shot 2의 start_image**.
3. Shot 2 끝 프레임 → Shot 3 start_image … 이렇게 4컷을 사슬처럼 잇는다.
4. 각 컷은 끝에 **"hero hold"(1초 정지감)** 를 둬서 마지막 프레임이 안정적이게 → 다음 컷 시작점으로 깨끗하게 쓰임.

> Seedance에 `end_image`도 넣을 수 있으면, 다음 컷 시작 이미지를 미리 정해 양쪽을 고정하면 더 매끈함.

---

## ⚙️ 공통 설정 (4컷 동일)

- **aspect_ratio:** `16:9` (노트북 풀스크린 촬영용)
- **duration:** `5` 초 (→ ffmpeg로 120프레임, 기존 car1~3과 동일)
- **resolution:** `720p` (기존 프레임 1280×720과 매칭 · 크레딧 절약)
- **mode:** `std`
- **generate_audio:** `false` (스크롤 프레임으로 쓰므로 무음)
- **genre:** `epic` 또는 `drama`

---

## 🎨 공통 스타일 앵커 (모든 프롬프트 앞에 그대로 붙이기)

```
Cinematic automotive studio film of a silver electric hypercar, dark glossy
reflective floor, soft volumetric haze, dramatic rim lighting with cool blue
and warm gold accents, photorealistic, ultra-detailed, shallow depth of field,
premium commercial aesthetic, 8k, consistent vehicle identity.
```

---

## 🎬 SHOT 1 — 분해 (Disassembly)

**start_image:** AXIS 히어로 풀카 정지컷
**카메라:** 느린 오비탈 + 살짝 푸시인

```
[STYLE ANCHOR]
The hypercar slowly disassembles into an elegant exploded view: body panels,
doors, wheels and hood detach and float gently outward, suspended in mid-air,
rotating slowly, revealing the engineering underneath. The camera performs a
slow orbit around the floating parts. Ends holding on the fully exploded car.
```

**끝 상태:** 차가 완전히 분해되어 공중에 떠 있는 구도 (= Shot 2 시작점)

---

## 🎬 SHOT 2 — 내부 줌 (Push into interior)

**start_image:** Shot 1의 마지막 프레임 (분해된 상태)
**카메라:** 정면 돌리 / FPV 푸시인

```
[STYLE ANCHOR]
The camera glides forward through the floating suspended parts, components
blur past on both sides, accelerating toward the open cabin, pushing through
the space where the door panel was, entering the interior. Ends at the cabin
threshold looking into the cockpit.
```

**끝 상태:** 캐빈 입구에서 운전석을 바라보는 구도 (= Shot 3 시작점)

---

## 🎬 SHOT 3 — 운전석 UI (Cockpit)

**start_image:** Shot 2의 마지막 프레임 (캐빈 입구)
**카메라:** 안착 + 느린 틸트

```
[STYLE ANCHOR]
The camera settles inside the cockpit, slowly revealing the driver seat and a
minimalist glowing digital dashboard, ambient blue UI light across the panels,
a steering yoke, premium stitched materials and brushed metal. Calm, luxurious.
Ends holding on a hero framing of the cockpit.
```

**끝 상태:** 운전석·대시보드 히어로 구도 (= Shot 4 시작점)

---

## 🎬 SHOT 4 — 엔진/배터리 구조 (X-ray cutaway)

**start_image:** Shot 3의 마지막 프레임 (운전석)
**카메라:** 풀백 + 하강 + X-ray 트랜지션

```
[STYLE ANCHOR]
The camera pulls back and descends through the cabin floor, transitioning into
a transparent X-ray cutaway revealing the battery pack and electric motor
beneath the chassis: glowing battery cells, copper windings, technical
schematic glow lines. Ends holding on a hero cutaway of the powertrain.
```

**끝 상태:** 엔진/배터리 컷어웨이 히어로 (피날레)

---

## 🔪 생성 후 — 프레임 추출 (ffmpeg)

각 컷 mp4를 받으면, car1~3과 똑같은 형식으로 120프레임 추출:

```bash
# Shot 1 → car4 (4컷 모두 dir만 바꿔서 반복)
ffmpeg -i shot1.mp4 -vf "fps=24,scale=1280:720" car4/f%03d.jpg
ffmpeg -i shot2.mp4 -vf "fps=24,scale=1280:720" car5/f%03d.jpg
ffmpeg -i shot3.mp4 -vf "fps=24,scale=1280:720" car6/f%03d.jpg
ffmpeg -i shot4.mp4 -vf "fps=24,scale=1280:720" car7/f%03d.jpg
```

> 5초 × 24fps = 120프레임. 프레임 수가 다르면 엔진 SETS의 `n` 값만 맞춰주면 됨.

---

## 🔌 엔진 연결 (car-cinematic.html · 한 곳만 손대면 끝)

이미 **드롭인 슬롯**을 비워뒀어요. 두 군데만:

**① `SETS` 배열 주석 해제:**
```js
{ dir:"car4/", n:120 }, // 분해
{ dir:"car5/", n:120 }, // 내부 줌
{ dir:"car6/", n:120 }, // 운전석
{ dir:"car7/", n:120 }, // 엔진/배터리
```

**② `#scroll` 안에 섹션 4칸 복제** (data-set만 3·4·5·6으로):
```html
<section class="act" data-set="3"><div class="pin">
  <div class="eyebrow" data-rev>Exploded</div>
  <h1 class="head" data-rev><span class="grad">Every part,<br>engineered.</span></h1>
</div></section>
<!-- data-set 4·5·6 동일 패턴으로 운전석·엔진 카피 -->
```

끝나면 car1~3 뒤에 신규 4컷이 자동으로 이어져 **총 7챕터**가 됩니다.

---

## ✅ 톤 일관성 체크 (4컷 생성 시)

- [ ] 차체 색·휠 디자인이 컷마다 흔들리지 않게 (start_image 체인 유지가 핵심)
- [ ] 배경: 어두운 반사 바닥 + 헤이즈 고정
- [ ] 조명: 블루(쿨) + 골드(웜) 림라이트 유지
- [ ] 각 컷 끝 1초 hero hold (다음 컷 시작 프레임 안정화)
