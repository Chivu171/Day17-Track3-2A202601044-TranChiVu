# Lab 17 Submission — Tran Chi Vu

## Cau 1: Layer quan trong nhat
Semantic (E06, E11) + episodic (E04, E05) cho so sánh rõ vs no-memory baseline (2/11). Long-term (E02, E03, E08, E09) cung cấp cross-session recall, bao gồm recency/conflict (E08: TypeScript thay thế Python cho BLUEBIRD-42). Short-term (E01, E10) chi duy trì trong current thread.

## Cau 2: Case retrieve nhieu token nhat
E07 (mixed): merged long-term (USER_SUMMARY) + semantic (PAYMENT-RULE-3), đầy đủ Python preference + Idempotency-Key. Latency cao nhất: 2109.6ms.

## Cau 3: Case mixed (E07) ket hop memory nao?
Long-term + semantic. Evidence: "Python" (preference) + "Idempotency-Key" (domain policy). Context Block tự động tổng hợp preference, semantic graph cung cấp policy.

## Cau 4: Token reduction + vi sao no-memory co giai thich yeu?
Memory-enabled: 14.2% reduction, 100% hit rate (11/11). No-memory: ~68% reduction nhưng chỉ 18.2% hit rate (2/11). No-memory tiết kiệm token bằng cách trả về rỗng — cheap nhưng incorrect.

## E08: Recency trong long-term
E08 PASS 1693.3ms. USER_SUMMARY reflect override: "TypeScript + NestJS for BLUEBIRD-42, Python not permitted" — recency/conflict rule hoạt động đúng.

## E10: Compaction trong short-term
E10 PASS 0.3ms. Buffer giữ 6 messages + 2 durable notes + 8 compactions. Constraint REVIEW-DEADLINE-1600 được giữ trong DURABLE_NOTES dù 8 lượt filler đã compact.

## Trade-off: Context Block (Zep) vs Redis+Qdrant
Context Block tự động assemble facts từ user graph — thông minh hơn Redis KV đơn giản. Zep cung cấp provenance, validity ranges, semantic graph tích hợp; Redis/Qdrant chỉ là baseline chi phí.

## Guardrail chong memory poisoning
1) Consent registry bắt buộc opt-in. 2) PII auto-redact. 3) User isolation: dùng đúng user_id. 4) Heartbeat read-only. 5) Conflict rule: mới override cũ, giữ provenance.