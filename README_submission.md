# Lab 17 Submission — Tran Chi Vu

## Cau 1: Layer quan trong nhat trong bo test nay
Semantic (E06, E11) va episodic (E04, E05) cho so sánh ro rệt vs no-memory baseline — chi có 2/11 case pass khi không có memory durable. Long-term (E02, E03, E08, E09) cung cấp cross-session preference recall, bao gồm cả recency/conflict handling (E08: TypeScript thay thế Python cho BLUEBIRD-42). Short-term (E01, E10) chi duy trì trong current thread.

## Cau 2: Case retrieve nhieu token nhat
E07 (mixed) với merged context kết hợp long-term (USER_SUMMARY) + semantic (PAYMENT-RULE-3), tổng hợp đầy đủ Python preference + Idempotency-Key policy. Latency cao nhất: 2109.6ms.

## Cau 3: Case mixed (E07) can ket hop memory nao?
Long-term + semantic. Evidence bắt buộc: "Python" (long-term preference) + "Idempotency-Key" (semantic knowledge page). Context Block tự động tổng hợp preference, semantic graph cung cấp domain policy.

## Cau 4: Token reduction so voi full source, vi sao no-memory co reduction cao nhung hit rate thap?
Memory-enabled: 14.2% reduction, 100% hit rate (11/11). No-memory: ~68% reduction nhưng chỉ 18.2% hit rate (2/11). No-memory "tiết kiệm" token bằng cách trả về rỗng — cheap nhưng incorrect.

## Trade-off: Context Block (Zep) vs Redis+Qdrant local
Context Block tự động assemble relevant facts từ Zep user graph dựa trên query — thông minh hơn Redis KV thông thường. Redis chỉ lưu KV đơn giản (profile, facts) với TTL; Qdrant làm vector search thủ công bằng HashingVectorizer. Zep cung cấp provenance, validity ranges, và semantic graph tích hợp sẵn — Redis/Qdrant chỉ là baseline để so sánh chi phí.

## Guardrail chong memory poisoning
1) Consent registry (data/consent.json) bắt buộc opt-in trước durable ingestion. 2) PII minimization tự động redact email/phone trong privacy_guard.py. 3) User isolation: mỗi long-term/episodic call dùng đúng user_id — không leak giữa minh-lab17 và lan-lab17 (verified E09). 4) Heartbeat chỉ read-only maintenance, không tự tạo instruction/quyền mới. 5) Conflict rule: thông tin mới override cũ, giữ provenance.