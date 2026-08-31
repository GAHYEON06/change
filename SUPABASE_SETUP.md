# 수정이 마음 체크인 · Supabase 설정 가이드

## 🔧 데이터베이스 테이블 생성

### 1단계: Supabase 대시보드 접속
https://supabase.com/dashboard에 접속해서 당신의 프로젝트를 열어주세요.

### 2단계: SQL 편집기에서 테이블 생성
좌측 "SQL Editor"에서 새 쿼리를 생성하고 아래 코드를 복사-붙여넣기 하세요:

```sql
CREATE TABLE responses (
  id BIGSERIAL PRIMARY KEY,
  total_score INT,
  normalized_score INT,
  category TEXT,
  answers INT[],
  created_at TIMESTAMP WITH TIME ZONE,
  inserted_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- RLS 정책: 누구나 읽기 가능, 모든 사용자가 삽입 가능 (익명)
ALTER TABLE responses ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Allow public read" ON responses
  FOR SELECT USING (true);

CREATE POLICY "Allow public insert" ON responses
  FOR INSERT WITH CHECK (true);
```

실행(Run) 버튼을 클릭하세요.

### 3단계: 테이블 확인
"Table Editor"에서 `responses` 테이블이 생성되었는지 확인하세요.

## 📊 저장되는 데이터

각 체크인마다 다음이 저장됩니다:
- `total_score` — 원점수 (0~30)
- `normalized_score` — 정규화 점수 (0~10)
- `category` — 결과 카테고리 (rest/emerging/sunny/rainbow)
- `answers` — 10개 문항의 답변 배열 [0~3, 0~3, ...]
- `created_at` — 응답 시간

## 📈 통계 보기

### 전체 참여자 수
```sql
SELECT COUNT(*) FROM responses;
```

### 카테고리별 분포
```sql
SELECT category, COUNT(*) as count
FROM responses
GROUP BY category
ORDER BY count DESC;
```

### 평균 점수
```sql
SELECT 
  AVG(normalized_score) as avg_score,
  MIN(normalized_score) as min_score,
  MAX(normalized_score) as max_score
FROM responses;
```

### 시간대별 참여자 (일일 집계)
```sql
SELECT 
  DATE(created_at) as date,
  COUNT(*) as participants,
  ROUND(AVG(normalized_score)::NUMERIC, 2) as avg_score
FROM responses
GROUP BY DATE(created_at)
ORDER BY date DESC;
```

## ⚠️ 보안 주의사항

✅ **현재 코드에 포함된 것:**
- Publishable Key (공개 가능)
- Supabase URL (공개 가능)

❌ **절대 포함되지 않는 것:**
- Secret/Private Key (절대 클라이언트에 노출 금지)
- 이것은 서버 환경에서만 사용합니다

## 🔄 실시간 업데이트

페이지를 새로고침할 때마다 "지난 축제에 __명이 참여했어요" 카운트가 업데이트됩니다.

라이브 대시보드를 원하면 Supabase 대시보드의 "Realtime"을 활성화할 수 있습니다.

## 🆘 트러블슈팅

**"Failed to connect" 에러**
- Supabase URL과 Publishable Key가 정확한지 확인하세요
- 프로젝트가 활성화되어 있는지 확인하세요

**데이터가 저장되지 않음**
- 브라우저 개발자 도구 콘솔을 확인하세요 (F12)
- RLS 정책이 제대로 설정되었는지 확인하세요

---

질문이 있으시면 Supabase 공식 문서를 확인해 주세요:
https://supabase.com/docs
