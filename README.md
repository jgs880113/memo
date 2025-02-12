CREATE TABLE TEST
(
    TMSTART String,
    OCCR_DT String,
    PKT_SEQ String,
    STRTITLE String,
    SIP String,
    DIP String,
    COUNT String,
    
    -- 기존 인덱스
    INDEX idx_dt OCCR_DT TYPE set(1000) GRANULARITY 4,
    INDEX idx_title STRTITLE TYPE set(1000) GRANULARITY 4,
    INDEX idx_tmstart_tm OCCR_DT, TMSTART TYPE set(1000) GRANULARITY 4,
    
    -- PKT_SEQ는 고유값이므로 ORDER BY 절에 포함하여 정렬 기준으로 사용
)
ENGINE = MergeTree
PARTITION BY toYYYYMMDD(toDateTime(OCCR_DT, 'YYYYMMDDHHmm'))  -- 일별 파티셔닝
ORDER BY (OCCR_DT, STRTITLE, PKT_SEQ, TMSTART);  -- PKT_SEQ를 포함시켜 정렬
