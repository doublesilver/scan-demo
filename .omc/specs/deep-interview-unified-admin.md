# Deep Interview Spec: 통합 어드민 시스템 (ERD + 아키텍처)

## Metadata
- Rounds: 6
- Final Ambiguity Score: 16%
- Type: brownfield
- Generated: 2026-03-23
- Threshold: 20%
- Status: PASSED

## Clarity Breakdown
| Dimension | Score | Weight | Weighted |
|-----------|-------|--------|----------|
| Goal Clarity | 0.90 | 35% | 0.315 |
| Constraint Clarity | 0.75 | 25% | 0.188 |
| Success Criteria | 0.85 | 25% | 0.213 |
| Context Clarity | 0.80 | 15% | 0.120 |
| **Total Clarity** | | | **0.836** |
| **Ambiguity** | | | **16.4%** |

## Goal
scan13.exe를 완전 대체하는 통합 어드민 시스템. PDA 바코드 스캐너 + 스트랩 제품 백과사전 + 재고관리를 하나의 웹 시스템으로 통합. 제품 마스터(디자인 596건) 아래에 SKU(바코드 11,821건)가 연결되는 2계층 구조.

## Constraints
- 배포: 기존 미니PC (Windows 11, Tailscale 100.125.17.60)
- 사용자: 대표(전체 권한) + 직원(조회/PDA 재고수정)
- 기술: FastAPI + SQLite + Pydantic v2 (기존 스택 유지)
- 기존 PDA 앱(Android Kotlin): 유지, API 엔드포인트만 변경
- NAS WebDAV 이미지 연동 유지
- 스마트스토어 연동: ERD에 자리만, Phase 3에서 구현

## Non-Goals
- 모바일 웹 앱 (Phase 1에서는 제외)
- 쿠팡 API 연동 (Phase 3 이후)
- 다국어 지원
- 복수 브랜드 관리 (스페이스쉴드 단일)

## Acceptance Criteria (Phase 1)
- [ ] 런칭 매트릭스: 기종 × 스트랩 교차표 표시, 셀 클릭 시 상세
- [ ] 제품 계보: 패밀리별 기종 파생 시각화
- [ ] PDA 재고 수정: 바코드 스캔 → 수량 입력 → DB 반영 → 이력 기록
- [ ] 초고속 검색: 제품명/재질/바코드 0.5초 이내
- [ ] scan13.exe 기능 100% 대체
- [ ] 제품 마스터 ↔ SKU 2계층 구조 동작

## 대표 확인 필요사항
1. scan SKU ↔ 구글시트 제품의 매핑 기준 (제품명? SKU코드? 바코드?)
2. scan13.exe 완전 대체 vs 공존 기간 필요 여부
3. 직원 계정 권한 범위 (조회만? 재고수정도?)
4. NAS 이미지 폴더 구조 (img/ vs real_image/ 통일 여부)

## Ontology (Key Entities)
| Entity | Type | Fields | Relationships |
|--------|------|--------|---------------|
| ProductMaster | core | id, name, brand, material, fastening, connector, size, launch_status, image_url | has_many SKU, belongs_to ProductFamily |
| SKU | core | id, product_master_id, sku_code, color, size_option, barcode, stock_quantity | belongs_to ProductMaster, has_many Barcode, has_many StockLog |
| Barcode | supporting | id, sku_id, barcode, barcode_type | belongs_to SKU |
| StockLog | supporting | id, sku_id, before_qty, after_qty, memo, updated_by | belongs_to SKU |
| ProductFamily | core | id, base_name, variant_count | has_many ProductMaster |
| LaunchMatrix | core | id, product_family_id, category, status | belongs_to ProductFamily |
| WatchModel | supporting | id, brand, model_name, watch_size | many_to_many SKU |
| ChannelPrice | supporting (Phase 3) | id, sku_id, channel, price, url | belongs_to SKU |
| StoreSync | supporting (Phase 3) | id, channel, last_sync, status | standalone |
| User | supporting | id, name, role, permissions | standalone |

## Phase Structure
- Phase 0: PDA MVP (300만, 완료)
- Phase 1: 통합 어드민 핵심 (450만, 즉시 착수)
- Phase 2: PDA 재고관리 + 통합 (250만)
- Phase 3: 스마트스토어 연동 (300만)
