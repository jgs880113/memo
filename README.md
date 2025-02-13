CREATE TABLE TEST
(
    TMSTART String,
    OCCR_DT String,
    PKT_SEQ String,  -- 고유값
    STRTITLE String,
    SIP String,
    DIP String,
    COUNT String,
    
    -- 기존 인덱스들
    INDEX idx_dt OCCR_DT TYPE set(1000) GRANULARITY 4,
    INDEX idx_title STRTITLE TYPE set(1000) GRANULARITY 4,
    
    -- 정렬 최적화 인덱스
    INDEX idx_tmstart_tm OCCR_DT, TMSTART TYPE set(1000) GRANULARITY 4,  
    INDEX idx_sort_order TMSTART, OCCR_DT, PKT_SEQ TYPE set(1000) GRANULARITY 4,  
    
    -- JOIN 최적화용 인덱스
    INDEX idx_sip SIP TYPE set(1000) GRANULARITY 4,  
    INDEX idx_dip DIP TYPE set(1000) GRANULARITY 4   
)
ENGINE = MergeTree
PARTITION BY toYYYYMMDD(toDateTime(OCCR_DT, 'YYYYMMDDHHmm'))  -- 일별 파티셔닝
ORDER BY (OCCR_DT, STRTITLE, PKT_SEQ, TMSTART);  -- PKT_SEQ를 포함하여 정렬 최적화
