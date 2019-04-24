INSERT INTO frelgas.m_frelgas_tran_effic_month -- 修改 
( ps_code, ps_name, ps_type, region_code, p_datetime, tran_rate, effic_rate, tran_effic_rate, remark, entry_time, output_count )

SELECT DISTINCT
e.ps_code AS ps_code,
b.ps_name AS ps_name,
l.obligate10 :: INT AS ps_type,
( SUBSTRING ( b.region_code || '', 1, 6 ) || '000000' ) :: BIGINT AS region_code,
( now() - INTERVAL '2 month' ) AS p_datetime,  -- 修改 
f.tranrate AS tran_rate,
f.efficrate AS effic_rate,
f.tranefficrate AS tran_effic_rate,
'' AS remark,
now(),
water.outputCount AS output_count 
FROM
	ods_mautomonitor.ods_m_automonitor_tran_effic e
	LEFT JOIN ods_bmain.ods_b_main_baseinfo b ON e.ps_code = b.ps_code
	LEFT JOIN ods_bmain.ods_b_main_laber l ON e.ps_code = l.ps_code 
	AND l.ps_code = b.ps_code
	LEFT JOIN (
SELECT
	h.ps_code AS ps_code,
	COUNT ( DISTINCT h.s_point_code ) AS outputcount 
FROM
	ods_bmain.ods_b_main_gasoutput h  -- 修改
WHERE
	h.status = 1 
GROUP BY
	h.ps_code 
	) water 
	
	ON e.ps_code = water.ps_code
	LEFT JOIN (
SELECT G
	.ps_code AS ps_code,
	round( AVG ( G.tran_rate ), 2 ) AS tranRate,
	round( AVG ( G.effic_rate ), 2 ) AS efficrate,
	round( AVG ( G.tran_rate ) * AVG ( G.effic_rate ) / 100, 2 ) AS tranefficrate 
FROM
	ods_mautomonitor.ods_m_automonitor_tran_effic G 
WHERE
	-- 修改 
	to_char( G.p_datetime, 'YYYY-MM-DD' ) LIKE'2019-04' || '%' 
GROUP BY
	G.ps_code 
	) f ON f.ps_code = e.ps_code 
WHERE
	-- 修改 
	to_char( e.p_datetime, 'YYYY-MM-DD' ) LIKE'2019-04' || '%' 
	AND e.ps_code IN (
SELECT
	ps_code as ps_code
FROM
	ods_bmain.ods_b_main_laber 
	-- 废水 
-- WHERE
-- 	obligate1::int = 1 
-- 	OR obligate2::INT = 1 
-- 	OR obligate3::int = 1 
-- 	OR obligate4::int = 1 
-- 	OR obligate5::int = 1 
	-- 废气 
WHERE
	obligate6::int = 1 
	OR obligate8::int = 1 
	)