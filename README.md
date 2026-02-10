# 박태훈 | 게임 클라이언트 프로그래머

> 1998년생, 군필 (의경 만기제대)
> 
> 📧 hunobas.dev@gmail.com

---

## 핵심 역량

### 1. 스팀 게임 출시 경험 — My Little Puppy (드림모션 인턴)

![그림5](https://github.com/user-attachments/assets/2282e694-733e-4fde-b230-28a5b101ba55)

38인 팀에서 3개월간 스팀 데모 출시에 참여했습니다. NPC 애니메이션 관련 버그 3건을 직접 추적·해결하고, 다국어 폰트 시스템 및 슈퍼점프 콘텐츠를 담당했습니다.

<details>
<summary><b>버그 수정 사례 1: 루트모션 회전이 프레임 컨디션에 따라 달라지는 문제</b></summary>

| 문제 상태 | 정상 상태 |
|------|---------|
| ![ezgif com-video-to-gif-converter (4)](https://github.com/user-attachments/assets/c038982c-4e66-4c04-a3c1-a4878d3d48c5) | ![ezgif com-video-to-gif-converter (5)](https://github.com/user-attachments/assets/f0d4974a-a54d-4b7d-870c-e7b6ad794d5c) |

**증상:** 게임 FPS가 낮으면 정상 회전, 정상 FPS에서는 회전이 중간에 멈춤

**원인 추적:**
1. 소수점 오차 가설 → 검증 결과 0.1도 미만으로 육안 차이와 무관
2. `ActorPosDir` 내부에 목표 각도로 선형 보간하는 로직 발견
3. 루트모션 각도와 `ActorPosDir` 보간이 중복 적용되고 있었음

**해결:** 루트모션 전용 위치/각도 업데이트 메서드를 분리하여 외부 보간 로직 제외

</details>

<details>
<summary><b>버그 수정 사례 2: 루트모션 종료 시 앞으로 튀어나가는 문제</b></summary>

| 문제 상태 | 정상 상태 |
|------|---------|
| ![02_-ezgif com-video-to-gif-converter (3)](https://github.com/user-attachments/assets/4e19abda-d7a0-41ea-a84e-c6ffce35ca67) | ![02_-ezgif com-video-to-gif-converter (4)](https://github.com/user-attachments/assets/53eb6482-83c2-4f4d-bbe5-2e215c2b7f15) |

**원인:** Walk → RootMotion → Idle 전환 시, Walk 상태의 `WalkToIdle` 속도값이 초기화되지 않고 Idle까지 전달됨

**해결:** 루트모션 상태머신 종료 시점에 Idle 블렌딩 속도를 0으로 초기화

</details>

<details>
<summary><b>버그 수정 사례 3: 컷씬 일시정지 시 NPC 위치가 튀는 문제</b></summary>

| 문제 상태 | 정상 상태 |
|------|---------|
| ![PlayableDirector_-ezgif com-video-to-gif-converter (1)](https://github.com/user-attachments/assets/436f5f08-5091-4b73-bb91-3871885ca447) | ![ezgif com-video-to-gif-converter (6)](https://github.com/user-attachments/assets/9bb6594b-afca-43f8-b29a-31fe61e320e3) |

**원인:** PlayableDirector가 액터 위치를 직접 수정하는데, 컷씬 모드는 `LateUpdate`에서 위치를 재조정하지만 일시정지 모드는 이 처리가 없었음

**해결:** 일시정지 모드에서도 이전 모드가 컷씬이면 `HandleGameActors()` 호출

</details>

- 🐶 [경력기술서 상세 | Notion](https://ethereal-judo-1f1.notion.site/My-Little-Puppy-1c6486e2cdb980fcbc33f487a01bd7fc)

---

### 2. 최적화 & 안정화 경험

#### Unity 렌더링 최적화 — 목성의 노래

400만 버텍스 + 300개 머터리얼 씬에서 프레임 불안정 문제를 해결했습니다.

<img width="1548" height="591" alt="최적화 전후" src="https://github.com/user-attachments/assets/6b35a453-6a45-4258-9635-3bcff6062e97" />

| 지표 | Before | After |
|------|--------|-------|
| Batches | 2,650 | 601 |
| FPS | 30~60 | 120+ |

**방법:** MeshBaker로 방 단위 텍스처 아틀라스 + 콤바인 메쉬, 오클루전 컬링 설정

- 📂 [MeshBaker 에디터 확장 코드](https://github.com/Hunobas/Song-Of-Jupitor/blob/main/Scripts/Editor/MB3_ApplyCombinedMaterialToSourceObjects.cs)
- 📜 [최적화 개발일지](https://velog.io/@po127992/목성의-노래-MeshBaker-최적화-삽질기-텍스처-아틀라스만-vs-콤바인-메쉬까지)

#### Unreal 메모리 최적화 — TOGU: Planet Survivors

![ObjectPooling](https://github.com/user-attachments/assets/25ce068f-e07b-4c4d-b2a3-726d83b97d55)

오브젝트 풀링 시스템을 구현하여 **GC 호출 빈도 80% 감소**, 적 100+ 동시 생존에서도 60 FPS 안정화를 달성했습니다.

- 📂 [오브젝트 풀 매니저 코드](https://github.com/Hunobas/Planet/blob/main/Source/Planet/System/ObjectPoolManagerComponent.h)

---

### 3. 데이터 구조 설계 & UI/UX 구현

#### DataAsset 기반 밸런싱 시스템

![PlayDemoGIF](https://github.com/user-attachments/assets/9257c9b1-f15a-492e-9788-a3118e2ce21c)

기획자가 코드 수정 없이 몬스터/아이템을 추가하고 밸런싱할 수 있도록 설계했습니다.

| 구성 요소 | 설명 |
|-----------|------|
| `UWaveConfigDataAsset` | 웨이브별 적 종류, 수, 타이밍 |
| `UEnemyDataAsset` | 적 스탯 (HP, 공격력, 이동속도) |
| `URewardManager` | MVC 패턴 기반 보상 선택 시스템 |

- 📄 [신규 무기/아이템 추가 가이드 | Notion](https://ethereal-judo-1f1.notion.site/223486e2cdb980c5a807f920ebad70a6)
- 📄 [신규 몬스터 추가 가이드 | Notion](https://ethereal-judo-1f1.notion.site/223486e2cdb98001869cef28bb9bfbb5)

#### UI 구현 경험

- **보상 선택 위젯** (MVC 패턴): [RewardSelectionWidget.h](https://github.com/Hunobas/Planet/blob/main/Source/Planet/UI/RewardSelectionWidget.h)
- **인게임 HUD**: 체력바, 경험치, 무기 쿨다운 표시
- **다국어 시스템**: CSV 기반 런타임 언어 전환

---

### 4. Git 협업 & 이슈 대응 경험

| 경험 | 내용 |
|------|------|
| **드림모션 인턴** | 38인 팀, 브랜치/PR/코드리뷰 프로세스 경험 |
| **PAGE25 팀 프로젝트** | 인디게임 팀 개발, 목성의 노래 (5인) |
| **크래프톤 정글** | [THE RATTUS](https://github.com/younggun339/jungleTwo) 팀 프로젝트, 아키텍처 설계 및 마이크로 회의 주도 |

---

## 프로젝트 요약

| 프로젝트 | 엔진 | 기간 | 규모 | 핵심 역할 |
|----------|------|------|------|-----------|
| [My Little Puppy](https://ethereal-judo-1f1.notion.site/My-Little-Puppy-1c6486e2cdb980fcbc33f487a01bd7fc) | Unity | 2025.01~03 | 38명 | 버그 수정, 다국어 폰트, 슈퍼점프 |
| [목성의 노래](https://github.com/Hunobas/Song-Of-Jupitor) | Unity | 2025.06~2026.01 | 5명 | 렌더링 최적화, 컷씬, 퍼즐 로직 |
| [TOGU: Planet Survivors](https://github.com/Hunobas/Planet) | Unreal 5.4 | 2025.04~06 | 1명 | 전체 아키텍처 설계 및 구현 |

---

## 📞 Contact

- 휴대폰: 010-3702-1279
- 이메일: hunobas.dev@gmail.com
- 블로그: [Velog](https://velog.io/@po127992/posts)
- GitHub: [Hunobas](https://github.com/hunobas)
