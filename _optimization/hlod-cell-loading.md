---
title: HLOD 설정 및 Cell 로딩 제어
category: World Partition
summary: 대규모 오픈월드에서 HLOD와 Cell 로딩 전략을 통해 드로우 콜 및 메모리 이슈를 개선한 작업 정리
---

## 개요

Unreal Engine 5.6의 World Partition 환경에서 HLOD(Hierarchical Level of Detail)와 Cell 로딩 제어를 적용해 대규모 럁 환경의 렌더링 성능을 개선한 사례 정리입니다.

## 문제 상황

- 오픈월드 맵 전체 로딩 시 GPU 메모리 8GB 초과
- 멀리 거리 오브젝트 드로우콜 발생
- Cell 코어 로딩 범위 최적화 필요

## 접근 방법

### 1. HLOD 레이어 설정

```ini
; WorldPartition HLOD 설정 예시
[/Script/Engine.HLODLayer]
LayerType=MeshMerge
TransitionScreenSize=0.01
```

- HLOD0: 중거리 (스크린 크기 0.01 미만)
- HLOD1: 장거리 (스크린 크기 0.001 미만)

### 2. Cell 로딩 범위 조정

| 파라미터 | 변경 전 | 변경 후 |
|---|---|---|
| Cell Size | 12800 | 25600 |
| Loading Range | 204800 | 153600 |
| HLOD Distance | 819200 | 614400 |

### 3. Streaming Source 활용

- Player 위치 기반으로 Cell 우선순위 동적 산정
- 로딩 스레쓰 `r.WorldPartition.RuntimeSpatialHash.UpdateFrequency` 조정

## 결과

- GPU 메모리: 8.2GB → 5.8GB (-29%)
- 드로우콜: 프레임당 3~5회 → 0~1회
- 스트리밍 스터터: 평균 120ms → 45ms

## 참고

- [UE5 World Partition 공식 문서](https://dev.epicgames.com/documentation/)
- 내부 콘솔 커맨드: `wp.Runtime.ToggleCellLoadingEnabled`
