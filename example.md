# The query execution time with cardinalities provided by REAL, NeuroCard

Query:

```
EXPLAIN ANALYZE SELECT COUNT(*) FROM title t, movie_companies mc, cast_info ci, movie_info mi, movie_info_idx mi_idx, movie_keyword mk WHERE t.id=mc.movie_id AND t.id=ci.movie_id AND t.id=mi.movie_id AND t.id=mi_idx.movie_id AND t.id=mk.movie_id AND mi_idx.info_type_id>107 AND ci.role_id=5 AND mk.keyword_id<131663 AND mi.info_type_id<11 AND mc.company_type_id=2 AND t.production_year>1901 AND t.production_year<1984;

```


## Execution with PostgreSQL native card

```
                                                                                         QUERY PLAN                                                                                          
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=1214651.01..1214651.02 rows=1 width=8) (actual time=13229.667..13229.696 rows=1 loops=1)
   ->  Hash Join  (cost=527618.16..1214650.93 rows=32 width=0) (actual time=11866.706..13149.942 rows=1590067 loops=1)
         Hash Cond: (ci.movie_id = t.id)
         ->  Seq Scan on cast_info ci  (cost=0.00..683910.90 rows=832413 width=4) (actual time=5075.578..5344.189 rows=898389 loops=1)
               Filter: (role_id = 5)
               Rows Removed by Filter: 35345955
         ->  Hash  (cost=527616.95..527616.95 rows=97 width=20) (actual time=6508.226..6508.245 rows=1398897 loops=1)
               Buckets: 131072 (originally 1024)  Batches: 512 (originally 1)  Memory Usage: 3329kB
               ->  Hash Join  (cost=229378.65..527616.95 rows=97 width=20) (actual time=2664.775..6088.751 rows=1398897 loops=1)
                     Hash Cond: (mi.movie_id = t.id)
                     ->  Seq Scan on movie_info mi  (cost=0.00..265642.62 rows=8691912 width=4) (actual time=152.363..2321.387 rows=8752291 loops=1)
                           Filter: (info_type_id < 11)
                           Rows Removed by Filter: 6083429
                     ->  Hash  (cost=229378.29..229378.29 rows=29 width=16) (actual time=2484.367..2484.383 rows=28638 loops=1)
                           Buckets: 32768 (originally 1024)  Batches: 1 (originally 1)  Memory Usage: 1599kB
                           ->  Hash Join  (cost=131423.80..229378.29 rows=29 width=16) (actual time=1388.861..2476.526 rows=28638 loops=1)
                                 Hash Cond: (mk.movie_id = t.id)
                                 ->  Seq Scan on movie_keyword mk  (cost=0.00..81003.12 rows=4520286 width=4) (actual time=0.046..811.951 rows=4521296 loops=1)
                                       Filter: (keyword_id < 131663)
                                       Rows Removed by Filter: 2634
                                 ->  Hash  (cost=131423.60..131423.60 rows=16 width=12) (actual time=1164.659..1164.671 rows=197 loops=1)
                                       Buckets: 1024  Batches: 1  Memory Usage: 17kB
                                       ->  Hash Join  (cost=79721.24..131423.60 rows=16 width=12) (actual time=749.642..1164.524 rows=197 loops=1)
                                             Hash Cond: (mc.movie_id = t.id)
                                             ->  Seq Scan on movie_companies mc  (cost=0.00..46718.11 rows=1329090 width=4) (actual time=0.258..408.834 rows=1334883 loops=1)
                                                   Filter: (company_type_id = 2)
                                                   Rows Removed by Filter: 1274246
                                             ->  Hash  (cost=79720.87..79720.87 rows=30 width=8) (actual time=615.810..615.815 rows=118 loops=1)
                                                   Buckets: 1024  Batches: 1  Memory Usage: 13kB
                                                   ->  Hash Join  (cost=24712.16..79720.87 rows=30 width=8) (actual time=285.746..615.721 rows=118 loops=1)
                                                         Hash Cond: (t.id = mi_idx.movie_id)
                                                         ->  Seq Scan on title t  (cost=0.00..51591.68 rows=546676 width=4) (actual time=0.031..376.56
3 rows=543218 loops=1)
                                                               Filter: ((production_year > 1901) AND (production_year < 1984))
                                                               Rows Removed by Filter: 1985094
                                                         ->  Hash  (cost=24710.44..24710.44 rows=138 width=4) (actual time=179.127..179.128 rows=260 l
oops=1)
                                                               Buckets: 1024  Batches: 1  Memory Usage: 18kB
                                                               ->  Seq Scan on movie_info_idx mi_idx  (cost=0.00..24710.44 rows=138 width=4) (actual t
ime=178.988..179.044 rows=260 loops=1)
                                                                     Filter: (info_type_id > 107)
                                                                     Rows Removed by Filter: 1379775
 Planning Time: 2.078 ms
 Execution Time: 13229.950 ms
(41 rows)
```



## Plan with REAL card

```
                                                                                        QUERY PLAN                                                     
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=1221114.44..1221114.45 rows=1 width=8) (actual time=5218.042..5218.051 rows=1 loops=1)
   ->  Hash Join  (cost=917540.53..1217139.27 rows=1590065 width=0) (actual time=3658.775..5163.497 rows=1590067 loops=1)
         Hash Cond: (mi.movie_id = t.id)
         ->  Seq Scan on movie_info mi  (cost=0.00..265642.62 rows=8752290 width=4) (actual time=68.460..1011.178 rows=8752291 loops=1)
               Filter: (info_type_id < 11)
               Rows Removed by Filter: 6083429
         ->  Hash  (cost=917130.68..917130.68 rows=32788 width=20) (actual time=3576.154..3576.161 rows=32788 loops=1)
               Buckets: 65536  Batches: 1  Memory Usage: 2178kB
               ->  Hash Join  (cost=229749.04..917130.68 rows=32788 width=20) (actual time=3373.335..3572.137 rows=32788 loops=1)
                     Hash Cond: (ci.movie_id = t.id)
                     ->  Seq Scan on cast_info ci  (cost=0.00..683910.90 rows=898392 width=4) (actual time=2270.172..2419.112 rows=898389 loops=1)
                           Filter: (role_id = 5)
                           Rows Removed by Filter: 35345955
                     ->  Hash  (cost=229391.04..229391.04 rows=28640 width=16) (actual time=1100.979..1100.985 rows=28638 loops=1)
                           Buckets: 32768  Batches: 1  Memory Usage: 1599kB
                           ->  Hash Join  (cost=131429.55..229391.04 rows=28640 width=16) (actual time=622.264..1097.574 rows=28638 loops=1)
                                 Hash Cond: (mk.movie_id = t.id)
                                 ->  Seq Scan on movie_keyword mk  (cost=0.00..81003.12 rows=4521296 width=4) (actual time=0.026..358.361 rows=4521296 loops=1)
                                       Filter: (keyword_id < 131663)
                                       Rows Removed by Filter: 2634
                                 ->  Hash  (cost=131427.10..131427.10 rows=196 width=12) (actual time=523.675..523.680 rows=197 loops=1)
                                       Buckets: 1024  Batches: 1  Memory Usage: 17kB
                                       ->  Hash Join  (cost=79702.54..131427.10 rows=196 width=12) (actual time=340.875..523.623 rows=197 loops=1)
                                             Hash Cond: (mc.movie_id = t.id)
                                             ->  Seq Scan on movie_companies mc  (cost=0.00..46718.11 rows=1334884 width=4) (actual time=0.123..183.616 rows=1334883 loops=1)
                                                   Filter: (company_type_id = 2)
                                                   Rows Removed by Filter: 1274246
                                             ->  Hash  (cost=79701.04..79701.04 rows=120 width=8) (actual time=274.396..274.400 rows=118 loops=1)
                                                   Buckets: 1024  Batches: 1  Memory Usage: 13kB
                                                   ->  Hash Join  (cost=24713.70..79701.04 rows=120 width=8) (actual time=126.914..274.359 rows=118 loops=1)
                                                         Hash Cond: (t.id = mi_idx.movie_id)
                                                         ->  Seq Scan on title t  (cost=0.00..51591.68 rows=543216 width=4) (actual time=0.015..168.242 rows=543218 loops=1)
                                                               Filter: ((production_year > 1901) AND (production_year < 1984))
                                                               Rows Removed by Filter: 1985094
                                                         ->  Hash  (cost=24710.44..24710.44 rows=261 width=4) (actual time=77.827..77.829 rows=260 loops=1)
                                                               Buckets: 1024  Batches: 1  Memory Usage: 18kB
                                                               ->  Seq Scan on movie_info_idx mi_idx  (cost=0.00..24710.44 rows=261 width=4) (actual time=77.756..77.781 rows=260 loops=1)
                                                                     Filter: (info_type_id > 107)
                                                                     Rows Removed by Filter: 1379775
 Planning Time: 43.641 ms
 Execution Time: 5219.961 ms
(41 rows)
```


## Plan with NeuroCard card

```

                                                                             QUERY PLAN                                                                
---------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=1199396.06..1199396.07 rows=1 width=8) (actual time=7346.527..7346.535 rows=1 loops=1)
   ->  Hash Join  (cost=911193.03..1188967.11 rows=4171581 width=0) (actual time=3970.143..7292.792 rows=1590067 loops=1)
         Hash Cond: (t.id = ci.movie_id)
         ->  Hash Join  (cost=104378.58..376160.92 rows=774197 width=12) (actual time=850.911..3885.180 rows=3506873 loops=1)
               Hash Cond: (mi.movie_id = t.id)
               ->  Seq Scan on movie_info mi  (cost=0.00..265642.62 rows=1539506 width=4) (actual time=69.631..1038.450 rows=8752291 loops=1)
                     Filter: (info_type_id < 11)
                     Rows Removed by Filter: 6083429
               ->  Hash  (cost=103626.05..103626.05 rows=60202 width=8) (actual time=780.275..780.277 rows=417232 loops=1)
                     Buckets: 131072 (originally 65536)  Batches: 8 (originally 1)  Memory Usage: 3073kB
                     ->  Hash Join  (cost=53366.59..103626.05 rows=60202 width=8) (actual time=240.468..730.461 rows=417232 loops=1)
                           Hash Cond: (mc.movie_id = t.id)
                           ->  Seq Scan on movie_companies mc  (cost=0.00..46718.11 rows=260031 width=4) (actual time=0.129..192.187 rows=1334883 loops=1)
                                 Filter: (company_type_id = 2)
                                 Rows Removed by Filter: 1274246
                           ->  Hash  (cost=51591.68..51591.68 rows=108153 width=4) (actual time=239.755..239.756 rows=543218 loops=1)
                                 Buckets: 131072 (originally 131072)  Batches: 8 (originally 2)  Memory Usage: 3403kB
                                 ->  Seq Scan on title t  (cost=0.00..51591.68 rows=108153 width=4) (actual time=0.013..179.182 rows=543218 loops=1)
                                       Filter: ((production_year > 1901) AND (production_year < 1984))
                                       Rows Removed by Filter: 1985094
         ->  Hash  (cost=806058.19..806058.19 rows=60501 width=12) (actual time=3094.415..3094.418 rows=41281 loops=1)
               Buckets: 65536  Batches: 1  Memory Usage: 2286kB
               ->  Hash Join  (cost=712295.97..806058.19 rows=60501 width=12) (actual time=2608.518..3089.702 rows=41281 loops=1)
                     Hash Cond: (mk.movie_id = ci.movie_id)
                     ->  Seq Scan on movie_keyword mk  (cost=0.00..81003.12 rows=3363945 width=4) (actual time=0.016..345.243 rows=4521296 loops=1)
                           Filter: (keyword_id < 131663)
                           Rows Removed by Filter: 2634
                     ->  Hash  (cost=712243.56..712243.56 rows=4193 width=8) (actual time=2509.311..2509.313 rows=275 loops=1)
                           Buckets: 8192  Batches: 1  Memory Usage: 75kB
                           ->  Hash Join  (cost=24948.19..712243.56 rows=4193 width=8) (actual time=2320.032..2509.238 rows=275 loops=1)
                                 Hash Cond: (ci.movie_id = mi_idx.movie_id)
                                 ->  Seq Scan on cast_info ci  (cost=0.00..683910.90 rows=525165 width=4) (actual time=2245.218..2391.712 rows=898389 loops=1)
                                       Filter: (role_id = 5)
                                       Rows Removed by Filter: 35345955
                                 ->  Hash  (cost=24710.44..24710.44 rows=19020 width=4) (actual time=72.837..72.838 rows=260 loops=1)
                                       Buckets: 32768  Batches: 1  Memory Usage: 266kB
                                       ->  Seq Scan on movie_info_idx mi_idx  (cost=0.00..24710.44 rows=19020 width=4) (actual time=72.766..72.790 rows=260 loops=1)
                                             Filter: (info_type_id > 107)
                                             Rows Removed by Filter: 1379775
 Planning Time: 43.682 ms
 Execution Time: 7348.264 ms
(41 rows)
```




## Plan with MIX card
```
                                                                                         QUERY PLAN                                                                                          
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=1189449.89..1189449.90 rows=1 width=8) (actual time=5203.722..5203.730 rows=1 loops=1)
   ->  Hash Join  (cost=1084772.76..1179020.94 rows=4171581 width=0) (actual time=4472.860..5146.699 rows=1590067 loops=1)
         Hash Cond: (mk.movie_id = t.id)
         ->  Seq Scan on movie_keyword mk  (cost=0.00..81003.12 rows=3363945 width=4) (actual time=0.167..361.492 rows=4521296 loops=1)
               Filter: (keyword_id < 131663)
               Rows Removed by Filter: 2634
         ->  Hash  (cost=1084180.63..1084180.63 rows=47370 width=20) (actual time=4371.383..4371.389 rows=10329 loops=1)
               Buckets: 65536  Batches: 1  Memory Usage: 1037kB
               ->  Hash Join  (cost=398210.98..1084180.63 rows=47370 width=20) (actual time=4173.219..4370.066 rows=10329 loops=1)
                     Hash Cond: (ci.movie_id = t.id)
                     ->  Seq Scan on cast_info ci  (cost=0.00..683910.90 rows=525165 width=4) (actual time=2223.152..2370.122 rows=898389 loops=1)
                           Filter: (role_id = 5)
                           Rows Removed by Filter: 35345955
                     ->  Hash  (cost=397673.03..397673.03 rows=43036 width=16) (actual time=1947.725..1947.730 rows=9062 loops=1)
                           Buckets: 65536  Batches: 1  Memory Usage: 937kB
                           ->  Hash Join  (cost=126242.83..397673.03 rows=43036 width=16) (actual time=594.788..1946.389 rows=9062 loops=1)
                                 Hash Cond: (mi.movie_id = t.id)
                                 ->  Seq Scan on movie_info mi  (cost=0.00..265642.62 rows=1539506 width=4) (actual time=67.018..1001.511 rows=8752291 loops=1)
                                       Filter: (info_type_id < 11)
                                       Rows Removed by Filter: 6083429
                                 ->  Hash  (cost=126213.22..126213.22 rows=2369 width=12) (actual time=515.615..515.619 rows=197 loops=1)
                                       Buckets: 4096  Batches: 1  Memory Usage: 41kB
                                       ->  Hash Join  (cost=73459.58..126213.22 rows=2369 width=12) (actual time=371.760..515.565 rows=197 loops=1)
                                             Hash Cond: (t.id = mc.movie_id)
                                             ->  Seq Scan on title t  (cost=0.00..51591.68 rows=308153 width=4) (actual time=0.011..165.821 rows=543218 loops=1)
                                                   Filter: ((production_year > 1901) AND (production_year < 1984))
                                                   Rows Removed by Filter: 1985094
                                             ->  Hash  (cost=73394.06..73394.06 rows=5241 width=8) (actual time=322.128..322.131 rows=737 loops=1)
                                                   Buckets: 8192  Batches: 1  Memory Usage: 93kB
                                                   ->  Hash Join  (cost=24948.19..73394.06 rows=5241 width=8) (actual time=140.348..321.998 rows=737 loops=1)
                                                         Hash Cond: (mc.movie_id = mi_idx.movie_id)
                                                         ->  Seq Scan on movie_companies mc  (cost=0.00..46718.11 rows=260031 width=4) (actual time=0.
105..181.931 rows=1334883 loops=1)
                                                               Filter: (company_type_id = 2)
                                                               Rows Removed by Filter: 1274246
                                                         ->  Hash  (cost=24710.44..24710.44 rows=19020 width=4) (actual time=72.689..72.690 rows=260 l
oops=1)
                                                               Buckets: 32768  Batches: 1  Memory Usage: 266kB
                                                               ->  Seq Scan on movie_info_idx mi_idx  (cost=0.00..24710.44 rows=19020 width=4) (actual
 time=72.621..72.649 rows=260 loops=1)
                                                                     Filter: (info_type_id > 107)
                                                                     Rows Removed by Filter: 1379775
 Planning Time: 44.779 ms
 Execution Time: 5204.808 ms
(41 rows)
```




