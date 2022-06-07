# The query execution time with cardinalities provided by REAL, NeuroCard
# Example 1
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





# Example 2

Query:

```
EXPLAIN ANALYZE SELECT COUNT(*) FROM title t,cast_info ci,movie_info mi,complete_cast cc,movie_info_idx mi_idx,movie_keyword mk,movie_companies mc WHERE t.id=mc.movie_id AND t.id=ci.movie_id AND t.id=mi.movie_id AND t.id=mi_idx.movie_id AND t.id=mk.movie_id AND t.id=cc.movie_id AND t.production_year>1986 AND t.production_year<1991 AND ci.role_id=9 AND mi.info_type_id=8 AND cc.status_id=3;
```

## Execution with PostgreSQL native card

```                                   
                                                                                              QUERY PLAN                        
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=619121.58..619121.59 rows=1 width=8) (actual time=3277.252..3277.267 rows=1 loops=1)
   ->  Hash Join  (cost=301126.60..619119.34 rows=898 width=0) (actual time=2011.105..3184.596 rows=2917445 loops=1)
         Hash Cond: (mi.movie_id = t.id)
         ->  Bitmap Heap Scan on movie_info mi  (cost=15029.70..327957.37 rows=1348292 width=4) (actual time=35.631..169.903 rows=1325361 loops=1)
               Recheck Cond: (info_type_id = 8)
               Heap Blocks: exact=7852
               ->  Bitmap Index Scan on info_type_id_movie_info  (cost=0.00..14692.62 rows=1348292 width=0) (actual time=34.781..34.781 rows=1325361 loops=1)
                     Index Cond: (info_type_id = 8)
         ->  Hash  (cost=286075.87..286075.87 rows=1682 width=24) (actual time=1973.126..1973.138 rows=2159219 loops=1)
               Buckets: 65536 (originally 2048)  Batches: 256 (originally 1)  Memory Usage: 3585kB
               ->  Hash Join  (cost=199401.02..286075.87 rows=1682 width=24) (actual time=996.682..1706.530 rows=2159219 loops=1)
                     Hash Cond: (mk.movie_id = t.id)
                     ->  Seq Scan on movie_keyword mk  (cost=0.00..69693.30 rows=4523930 width=4) (actual time=0.015..256.320 rows=4523930 loops=1)
                     ->  Hash  (cost=199389.27..199389.27 rows=940 width=20) (actual time=996.403..996.414 rows=64147 loops=1)
                           Buckets: 65536 (originally 1024)  Batches: 1 (originally 1)  Memory Usage: 3770kB
                           ->  Hash Join  (cost=144714.34..199389.27 rows=940 width=20) (actual time=652.485..988.212 rows=64147 loops=1)
                                 Hash Cond: (mc.movie_id = t.id)
                                 ->  Seq Scan on movie_companies mc  (cost=0.00..44881.29 rows=2609129 width=4) (actual time=0.011..158.901 rows=2609129 loops=1)
                                 ->  Hash  (cost=144702.95..144702.95 rows=911 width=16) (actual time=652.194..652.202 rows=12484 loops=1)
                                       Buckets: 16384 (originally 1024)  Batches: 1 (originally 1)  Memory Usage: 714kB
                                       ->  Hash Join  (cost=106646.57..144702.95 rows=911 width=16) (actual time=453.673..650.504 rows=12484 loops=1)
                                             Hash Cond: (ci.movie_id = t.id)
                                             ->  Index Scan using role_id_cast_info on cast_info ci  (cost=0.44..33533.91 rows=1203684 width=4) (actual  time=0.012..118.665 rows=1093558 loops=1)
                                                   Index Cond: (role_id = 9)
                                             ->  Hash  (cost=106622.23..106622.23 rows=1912 width=12) (actual time=453.263..453.270 rows=11890 loops=1)
                                                   Buckets: 16384 (originally 2048)  Batches: 1 (originally 1)  Memory Usage: 639kB
                                                   ->  Hash Join  (cost=79692.63..106622.23 rows=1912 width=12) (actual time=283.157..451.737 rows=11890 loops=1)
                                                         Hash Cond: (mi_idx.movie_id = t.id)
                                                         ->  Seq Scan on movie_info_idx mi_idx  (cost=0.00..21735.35 rows=1380035 width=4) (actual time=0.010..78.701 rows=1380035 loops=1)
                                                         ->  Hash  (cost=79648.84..79648.84 rows=3503 width=8) (actual time=283.119..283.123 rows=5846 loops=1)
                                                               Buckets: 8192 (originally 4096)  Batches: 1 (originally 1)  Memory Usage: 293kB
                                                               ->  Hash Join  (cost=4233.00..79648.84 rows=3503 width=8) (actual time=24.709..282.257 rows=5846 loops=1)
                                                                     Hash Cond: (t.id = cc.movie_id)
                                                                     ->  Seq Scan on title t  (cost=0.00..73920.11 rows=80139 width=4) (actual time=0.014..238.357 rows=84335 loops=1)
                                                                           Filter: ((production_year > 1986) AND (production_year < 1991))
                                                                           Rows Removed by Filter: 2443977
                                                                     ->  Hash  (cost=2419.57..2419.57 rows=110514 width=4) (actual time=24.252..24.254 rows=110494 loops=1)
                                                                           Buckets: 131072  Batches: 2  Memory Usage: 2957kB
                                                                           ->  Seq Scan on complete_cast cc  (cost=0.00..2419.57 rows=110514 width=4) (actual time=0.004..13.247 rows=110494 loops=1)
                                                                                 Filter: (status_id = 3)
                                                                                 Rows Removed by Filter: 24592
 Planning Time: 20.423 ms
 Execution Time: 3278.159 ms
(43 rows)
```



## Execution with PostgreSQL Neuro card
``` 
                                                                                  QUERY PLAN                                          
---------------------------------------------------------------------------------------------------------------------------------------------------------------------- -------
 Aggregate  (cost=629865.62..629865.63 rows=1 width=8) (actual time=2728.007..2728.015 rows=1 loops=1)
   ->  Hash Join  (cost=528474.55..625111.91 rows=1901484 width=0) (actual time=1785.996..2637.278 rows=2917445 loops=1)
         Hash Cond: (t.id = mc.movie_id)
         ->  Hash Join  (cost=133150.88..225512.06 rows=255526 width=16) (actual time=629.354..1179.972 rows=534230 loops=1)               
         Hash Cond: (mk.movie_id = t.id)
               ->  Seq Scan on movie_keyword mk  (cost=0.00..69693.30 rows=4523930 width=4) (actual time=0.029..230.093 rows=4523930 loops=1)                       
               ->  Hash  (cost=133117.19..133117.19 rows=2695 width=12) (actual time=629.273..629.276 rows=46044 loops=1)
                     Buckets: 65536 (originally 4096)  Batches: 1 (originally 1)  Memory Usage: 2491kB                    
                     ->  Hash Join  (cost=99333.52..133117.19 rows=2695 width=12) (actual time=406.340..623.459 rows=46044 loops=1)
                           Hash Cond: (ci.movie_id = t.id)                           
                           ->  Index Scan using role_id_cast_info on cast_info ci  (cost=0.44..33533.91 rows=39950 width=4) (actual time=0.033..118.896 rows=1093558 loops=1)
                                 Index Cond: (role_id = 9)                           
                                 ->  Hash  (cost=99292.98..99292.98 rows=3208 width=8) (actual time=406.264..406.266 rows=64655 loops=1)
                                 Buckets: 65536 (originally 4096)  Batches: 1 (originally 1)  Memory Usage: 3038kB
                                 ->  Hash Join  (cost=73935.03..99292.98 rows=3208 width=8) (actual time=222.052..399.109 rows=64655 loops=1)
                                       Hash Cond: (mi_idx.movie_id = t.id)
                                       ->  Seq Scan on movie_info_idx mi_idx  (cost=0.00..21735.35 rows=1380035 width=4) (actual time=0.015..70.138 rows=1380035 loops=1)
                                       ->  Hash  (cost=73920.11..73920.11 rows=1193 width=4) (actual time=222.007..222.007 rows=84335 loops=1)
                                             Buckets: 131072 (originally 2048)  Batches: 1 (originally 1)  Memory Usage: 3989kB
                                             ->  Seq Scan on title t  (cost=0.00..73920.11 rows=1193 width=4) (actual time=0.013..211.193 rows=84335 loops=1)
                                                   Filter: ((production_year > 1986) AND (production_year < 1991))
                                                   Rows Removed by Filter: 2443977
         ->  Hash  (cost=392899.42..392899.42 rows=139460 width=12) (actual time=1154.644..1154.648 rows=451940 loops=1)
               Buckets: 131072 (originally 131072)  Batches: 8 (originally 4)  Memory Usage: 3404kB
               ->  Hash Join  (cost=78762.77..392899.42 rows=139460 width=12) (actual time=702.961..1105.964 rows=451940 loops=1)
                     Hash Cond: (mi.movie_id = mc.movie_id)
                     ->  Bitmap Heap Scan on movie_info mi  (cost=14701.56..327629.24 rows=35753 width=4) (actual time=29.097..181.402 rows=1325361 loops=1)
                           Recheck Cond: (info_type_id = 8)
                           Heap Blocks: exact=7852
                           ->  Bitmap Index Scan on info_type_id_movie_info  (cost=0.00..14692.62 rows=1348292 width=0) (actual time=28.248..28.248 rows=1325361 loops=1)
                                 Index Cond: (info_type_id = 8)
                     ->  Hash  (cost=61230.64..61230.64 rows=172525 width=8) (actual time=673.381..673.383 rows=384956 loops=1)
                           Buckets: 131072 (originally 131072)  Batches: 8 (originally 4)  Memory Usage: 3073kB
                           ->  Hash Join  (cost=2593.22..61230.64 rows=172525 width=8) (actual time=23.832..631.221 rows=384956 loops=1)
                                 Hash Cond: (mc.movie_id = cc.movie_id)
                                 ->  Seq Scan on movie_companies mc  (cost=0.00..44881.29 rows=2609129 width=4) (actual time=0.013..146.554 rows=2609129 loops=1)
                                 ->  Hash  (cost=2419.57..2419.57 rows=13892 width=4) (actual time=23.618..23.619 rows=110494 loops=1)
                                       Buckets: 131072 (originally 16384)  Batches: 2 (originally 1)  Memory Usage: 3073kB
                                       ->  Seq Scan on complete_cast cc  (cost=0.00..2419.57 rows=13892 width=4) (actual time=0.012..11.320 rows=110494 loops=1)
                                             Filter: (status_id = 3)
                                             Rows Removed by Filter: 24592
 Planning Time: 2450.745 ms
 Execution Time: 2729.512 ms
```     


## Execution with PostgreSQL MIX card
``` 
                                                                                              QUERY PLAN                                                                                              
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=622732.72..622732.73 rows=1 width=8) (actual time=2367.568..2367.624 rows=1 loops=1)
   ->  Hash Join  (cost=531311.99..617979.01 rows=1901484 width=0) (actual time=1300.812..2273.522 rows=2917445 loops=1)
         Hash Cond: (mk.movie_id = t.id)
         ->  Seq Scan on movie_keyword mk  (cost=0.00..69693.30 rows=4523930 width=4) (actual time=0.025..248.447 rows=4523930 loops=1)
         ->  Hash  (cost=531305.72..531305.72 rows=502 width=24) (actual time=1299.361..1299.413 rows=83721 loops=1)
               Buckets: 65536 (originally 1024)  Batches: 2 (originally 1)  Memory Usage: 3585kB
               ->  Hash Join  (cost=213991.01..531305.72 rows=502 width=24) (actual time=1064.112..1288.198 rows=83721 loops=1)
                     Hash Cond: (mi.movie_id = t.id)
                     ->  Bitmap Heap Scan on movie_info mi  (cost=14556.83..326810.43 rows=1348292 width=4) (actual time=30.446..136.001 rows=1325361 loops=1)
                           Recheck Cond: (info_type_id = 8)
                           Heap Blocks: exact=7852
                           ->  Bitmap Index Scan on info_type_id_movie_info  (cost=0.00..14219.75 rows=1303376 width=0) (actual time=29.554..29.557 rows=1325361 loops=1)
                                 Index Cond: (info_type_id = 8)
                     ->  Hash  (cost=199422.43..199422.43 rows=940 width=20) (actual time=1033.454..1033.492 rows=64147 loops=1)
                           Buckets: 65536 (originally 1024)  Batches: 1 (originally 1)  Memory Usage: 3770kB
                           ->  Hash Join  (cost=144747.51..199422.43 rows=940 width=20) (actual time=659.597..1024.691 rows=64147 loops=1)
                                 Hash Cond: (mc.movie_id = t.id)
                                 ->  Seq Scan on movie_companies mc  (cost=0.00..44881.29 rows=2609129 width=4) (actual time=0.017..173.977 rows=2609129 loops=1)
                                 ->  Hash  (cost=144736.12..144736.12 rows=911 width=16) (actual time=659.327..659.359 rows=12484 loops=1)
                                       Buckets: 16384 (originally 1024)  Batches: 1 (originally 1)  Memory Usage: 714kB
                                       ->  Hash Join  (cost=106671.94..144736.12 rows=911 width=16) (actual time=441.514..657.629 rows=12484 loops=1)
                                             Hash Cond: (ci.movie_id = t.id)
                                             ->  Index Scan using role_id_cast_info on cast_info ci  (cost=0.44..33533.91 rows=1203684 width=4) (actual time=0.031..137.541 rows=1093558 loops=1)
                                                   Index Cond: (role_id = 9)
                                             ->  Hash  (cost=106627.13..106627.13 rows=3549 width=12) (actual time=441.032..441.058 rows=11890 loops=1)
                                                   Buckets: 16384 (originally 4096)  Batches: 1 (originally 1)  Memory Usage: 639kB
                                                   ->  Hash Join  (cost=79697.53..106627.13 rows=3549 width=12) (actual time=272.750..439.454 rows=11890 loops=1)
                                                         Hash Cond: (mi_idx.movie_id = t.id)
                                                         ->  Seq Scan on movie_info_idx mi_idx  (cost=0.00..21735.35 rows=1380035 width=4) (actual time=0.009..76.361 rows=1380035 loops=1)
                                                         ->  Hash  (cost=79653.74..79653.74 rows=3503 width=8) (actual time=272.710..272.726 rows=5846 loops=1)
                                                               Buckets: 8192 (originally 4096)  Batches: 1 (originally 1)  Memory Usage: 293kB
                                                               ->  Hash Join  (cost=4233.00..79653.74 rows=3503 width=8) (actual time=27.482..271.846 rows=5846 loops=1)
                                                                     Hash Cond: (t.id = cc.movie_id)
                                                                     ->  Seq Scan on title t  (cost=0.00..73925.02 rows=80139 width=4) (actual time=0.014..225.013 rows=84335 loops=1)
                                                                           Filter: ((production_year > 1986) AND (production_year < 1991))
                                                                           Rows Removed by Filter: 2443977
                                                                     ->  Hash  (cost=2419.57..2419.57 rows=110514 width=4) (actual time=26.946..26.952 rows=110494 loops=1)
                                                                           Buckets: 131072  Batches: 2  Memory Usage: 2957kB
                                                                           ->  Seq Scan on complete_cast cc  (cost=0.00..2419.57 rows=110514 width=4) (actual time=0.007..14.897 rows=110494 loops=1)
                                                                                 Filter: (status_id = 3)
                                                                                 Rows Removed by Filter: 24592

 Planning Time: 180.245 ms
 Execution Time: 2270.332 ms
```     



## Execution with PostgreSQL Real card
``` 

                                                                                           QUERY PLAN                                 
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Aggregate  (cost=653425.51..653425.52 rows=1 width=8) (actual time=1953.490..1953.497 rows=1 loops=1)
   ->  Merge Join  (cost=478680.53..646131.90 rows=2917445 width=0) (actual time=864.187..1858.501 rows=2917445 loops=1)
         Merge Cond: (t.id = mk.movie_id)
         ->  Merge Join  (cost=478664.97..545102.09 rows=83721 width=24) (actual time=864.008..1129.288 rows=83721 loops=1)               Merge Cond: (mc.movie_id = t.id)
               ->  Index Only Scan using movie_id_movie_companies on movie_companies mc  (cost=0.43..59697.37 rows=2609129 width=4) (actual time=0.010..165.614 rows=2608026 loops=1)                     Heap Fetches: 0
               ->  Sort  (cost=478664.54..478699.65 rows=14046 width=20) (actual time=863.672..867.275 rows=84234 loops=1)
                     Sort Key: t.id                     Sort Method: quicksort  Memory: 1482kB
                     ->  Hash Join  (cost=159733.69..477696.92 rows=14046 width=20) (actual time=650.630..861.295 rows=14046 loops=1)
                           Hash Cond: (mi.movie_id = t.id)                           ->  Bitmap Heap Scan on movie_info mi  (cost=15023.97..327951.64 rows=1325361 width=4) (actual time=28.793..146.838 rows=1325361 loops=1)
                                 Recheck Cond: (info_type_id = 8)                                 Heap Blocks: exact=7852
                                 ->  Bitmap Index Scan on info_type_id_movie_info  (cost=0.00..14692.62 rows=1348292 width=0) (actual time=27.951..27.951 rows=1325361 loops=1)
                                       Index Cond: (info_type_id = 8)                           ->  Hash  (cost=144553.67..144553.67 rows=12484 width=16) (actual time=621.769..621.774 rows=12484 loops=1)
                                 Buckets: 16384  Batches: 1  Memory Usage: 714kB
                                 ->  Hash Join  (cost=106867.93..144553.67 rows=12484 width=16) (actual time=421.748..620.002 rows=12484 loops=1)
                                       Hash Cond: (ci.movie_id = t.id)
                                       ->  Index Scan using role_id_cast_info on cast_info ci  (cost=0.44..33533.91 rows=1093558 width=4) (actual time=0.028..116.234 rows=1093558 loops=1)
                                             Index Cond: (role_id = 9)
                                       ->  Hash  (cost=106718.87..106718.87 rows=11890 width=12) (actual time=421.152..421.155 rows=11890 loops=1)
                                             Buckets: 16384  Batches: 1  Memory Usage: 639kB
                                             ->  Hash Join  (cost=79776.47..106718.87 rows=11890 width=12) (actual time=257.226..419.585 rows=11890 loops=1)
                                                   Hash Cond: (mi_idx.movie_id = t.id)
                                                   ->  Seq Scan on movie_info_idx mi_idx  (cost=0.00..21735.35 rows=1380035 width=4) (actual time=0.013..71.828 rows=1380035 loops=1)
                                                   ->  Hash  (cost=79703.40..79703.40 rows=5846 width=8) (actual time=257.175..257.177 rows=5846 loops=1)
                                                         Buckets: 8192  Batches: 1  Memory Usage: 293kB
                                                         ->  Hash Join  (cost=4232.75..79703.40 rows=5846 width=8) (actual time=23.950..256.282 rows=5846 loops=1)
                                                               Hash Cond: (t.id = cc.movie_id)
                                                               ->  Seq Scan on title t  (cost=0.00..73920.11 rows=84335 width=4) (actual time=0.011..213.194 rows=84335 loops=1)
                                                                     Filter: ((production_year > 1986) AND (production_year < 1991))
                                                                     Rows Removed by Filter: 2443977
                                                               ->  Hash  (cost=2419.57..2419.57 rows=110494 width=4) (actual time=23.500..23.501 rows=110494 loops=1)
                                                                     Buckets: 131072  Batches: 2  Memory Usage: 2957kB
                                                                     ->  Seq Scan on complete_cast cc  (cost=0.00..2419.57 rows=110494 width=4) (actual time=0.006..12.437 rows=110494 loops=1)
                                                                           Filter: (status_id = 3)
                                                                           Rows Removed by Filter: 24592
         ->  Index Only Scan using movie_id_movie_keyword on movie_keyword mk  (cost=0.43..88087.38 rows=4523930 width=4) (actual time=0.013..395.332 rows=7397208 loops=1)
               Heap Fetches: 0
 Execution Time: 1955.062 ms
``` 
