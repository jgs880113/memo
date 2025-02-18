
CREATE TABLE TEST
(
    OCCR_DT String,
    PKT_SEQ String, 
    TMSTART String,
    STRTITLE String,
    SIP String,
    DIP String,
    COUNT String,

    INDEX idx_dt OCCR_DT TYPE set(1000) GRANULARITY 4,
    INDEX idx_title STRTITLE TYPE set(1000) GRANULARITY 4,
    INDEX idx_sip SIP TYPE set(1000) GRANULARITY 4,
    INDEX idx_dip DIP TYPE set(1000) GRANULARITY 4
)
ENGINE = MergeTree
PARTITION BY toYYYYMMDD(toDateTime(OCCR_DT, 'Asia/Seoul')) 
ORDER BY (OCCR_DT, STRTITLE, PKT_SEQ, TMSTART);



ALTER TABLE test ADD INDEX idx_sip (SIP) TYPE set(1000) GRANULARITY 4;
ALTER TABLE test MATERIALIZE INDEX idx_sip;
OPTIMIZE TABLE test FINAL;
SET max_threads = 8;


CREATE TABLE TEST_IP_JOIN 
ENGINE = Join(ANY, LEFT, IP String, TITLE String)
AS SELECT IP, TITLE FROM TEST_IP WHERE USE_YN = 'Y';


WITH T AS (
	SELECT T.OCCR_DT, T.PKT_SEQ, T.TMSTART, T.SIP, T.DIP,
	       ROW_NUMBER() OVER(ORDER BY T.OCCR_DT DESC, T.STRTITLE DESC, T.PKT_SEQ DESC, T.TMSTART DESC) AS RNK
  	FROM TEST T
  	LEFT JOIN GLOBAL TEST_IP_JOIN IT 
           ON T.SIP = IT.IP 
          AND T.STRTITLE = IT.TITLE
  	LEFT JOIN GLOBAL TEST_IP_JOIN IT2 
           ON T.DIP = IT2.IP 
          AND T.STRTITLE = IT2.TITLE
  	WHERE T.OCCR_DT LIKE '202412%' 
	  AND T.OCCR_DT >= '202412010000' 
	  AND T.OCCR_DT <= '202412110000'
  	  AND T.STRTITLE = 'ATTACK'
)
SELECT T.OCCR_DT, T.PKT_SEQ, T.TMSTART, T.SIP, T.DIP, T.RNK
FROM T
ORDER BY T.OCCR_DT DESC, T.STRTITLE DESC, T.PKT_SEQ DESC, T.TMSTART DESC
LIMIT 100000;  -- 필요에 따라 조정
✅ 최적화 결과 요약
최적화 방법	성능 향상 효과
LIKE '202412%'로 OCCR_DT 필터링 최적화	파티션 스캔 범위 축소
테이블 ORDER BY와 정렬 순서 일치	정렬 비용 감소
GLOBAL JOIN 적용	JOIN 속도 향상
LIMIT 적용	불필요한 데이터 조회 방지
필요한 컬럼만 조회	네트워크 트래픽 절감
이렇게 변경하면 쿼리 성능이 크게 향상될 것입니다! 🚀

arrayJoin()?






