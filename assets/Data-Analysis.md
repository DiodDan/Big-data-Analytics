### [Back to readme](../README.md)

# Data Analysis

## We will start from this question:

#### `What is the general outlook on RTX 4060 GPU, the whole generation of the chip and the company?`

---

Here you can find sql queries for getting values from data

```postgresql
WITH sentiment_sums AS (SELECT SUM(CASE WHEN sentiment = 0 THEN likes ELSE 0 END)              AS neg_likes_total,
                               SUM(CASE WHEN sentiment = 1 THEN likes ELSE 0 END)              AS neu_likes_total,
                               SUM(CASE WHEN sentiment = 2 THEN likes ELSE 0 END)              AS pos_likes_total,
                               SUM(CASE WHEN sentiment = 0 THEN views ELSE 0 END)              AS neg_views_total,
                               SUM(CASE WHEN sentiment = 1 THEN views ELSE 0 END)              AS neu_views_total,
                               SUM(CASE WHEN sentiment = 2 THEN views ELSE 0 END)              AS pos_views_total,
                               SUM(CASE WHEN sentiment = 0 THEN comments + reposts ELSE 0 END) AS neg_shares_total,
                               SUM(CASE WHEN sentiment = 1 THEN comments + reposts ELSE 0 END) AS neu_shares_total,
                               SUM(CASE WHEN sentiment = 2 THEN comments + reposts ELSE 0 END) AS pos_shares_total,
                               COUNT(*)                                                        AS total_amount_of_posts
                        FROM processed_nvidia), -- Name of the table should be changed hor two observations
     weights AS (SELECT CAST(neg_likes_total AS DOUBLE PRECISION) / neg_views_total  AS neg_likes_per_view,
                        CAST(neu_likes_total AS DOUBLE PRECISION) / neu_views_total  AS neu_likes_per_view,
                        CAST(pos_likes_total AS DOUBLE PRECISION) / pos_views_total  AS pos_likes_per_view,
                        CAST(neg_likes_total AS DOUBLE PRECISION) / neg_shares_total AS neg_likes_per_share,
                        CAST(neu_likes_total AS DOUBLE PRECISION) / neu_shares_total AS neu_likes_per_share,
                        CAST(pos_likes_total AS DOUBLE PRECISION) / pos_shares_total AS pos_likes_per_share
                 FROM sentiment_sums),
     totals AS (SELECT weights.neg_likes_per_view * sentiment_sums.neg_views_total + 1 * sentiment_sums.neg_likes_total +
                       sentiment_sums.neg_shares_total * weights.neg_likes_per_share AS neg_total,
                       weights.neu_likes_per_view * sentiment_sums.neu_views_total + 1 * sentiment_sums.neu_likes_total +
                       sentiment_sums.neu_shares_total * weights.neu_likes_per_share AS neu_total,
                       weights.pos_likes_per_view * sentiment_sums.pos_views_total + 1 * sentiment_sums.pos_likes_total +
                       sentiment_sums.pos_shares_total * weights.pos_likes_per_share AS pos_total
                FROM sentiment_sums,
                     weights)
SELECT totals.neg_total / sentiment_sums.total_amount_of_posts AS neg_avg,
       totals.neu_total / sentiment_sums.total_amount_of_posts AS neu_avg,
       totals.pos_total / sentiment_sums.total_amount_of_posts AS pos_avg,
       totals.neg_total,
       totals.neu_total,
       totals.pos_total
FROM sentiment_sums,
     weights,
     totals;
 ```

| Measurement | neg_total | neu_total | pos_total | neg_avg           | neu_avg            | pos_avg           |
|-------------|-----------|-----------|-----------|-------------------|--------------------|-------------------|
| Competitors | 82140     | 127239    | 229482    | 33.78856437679967 | 52.340189222542165 | 94.39819004524887 |
| Nvidia      | 142338    | 1124049   | 1702794   | 36.86557886557887 | 291.1289821289821  | 441.024087024087  |

---

## Now we can go to another question:

#### `What countries were the most involved in the discussion?`

```postgresql
SELECT SUM(CAST(content ILIKE '%naira%' OR content ILIKE '%ngn%' OR content ILIKE '%nigeria%' AS INTEGER))    AS nigeria_amount,
       SUM(CAST(content ILIKE '%kes%' OR content ILIKE '%kenya%' AS INTEGER))                                 AS kenya_amount,
       SUM(CAST(content ILIKE '%euro%' OR content ILIKE '%eur%' AS INTEGER))                                  AS europe_amount,
       SUM(CAST(content ILIKE '%dollar%' OR content ILIKE '%usd%' OR content ILIKE '%us dollar%' AS INTEGER)) AS us_amount,
       SUM(CAST(content ILIKE '%gbp%' OR content ILIKE '%pounds%' AS INTEGER))                                AS gbp_amount,
       SUM(CAST(content ILIKE '%chf%' OR content ILIKE '%franc%' AS INTEGER))                                 AS swiss_amount,
       SUM(CAST(content ILIKE '%ghana%' OR content ILIKE '%ghc%' AS INTEGER))                                 AS ghana_amount,
       SUM(CAST(content ILIKE '%rupee%' OR content ILIKE '%inr%' AS INTEGER))                                 AS india_amount,
       SUM(CAST(content ILIKE '%cny%' OR content ILIKE '%yuan%' AS INTEGER))                                  AS china_amount,
       SUM(CAST(content ILIKE '%jpy%' OR content ILIKE '%yen%' AS INTEGER))                                   AS japan_amount

FROM processed_competitors; -- This should be changed to table name on different observations
```

| Measurement | nigeria | kenya | europe | usa | gdp | swiss | ghana | india | china | japan |
|-------------|---------|-------|--------|-----|-----|-------|-------|-------|-------|-------|
| Competitors | 24      | 58    | 34     | 9   | 15  | 4     | 40    | 2     | 0     | 0     |
| Nvidia      | 461     | 78    | 23     | 23  | 15  | 8     | 7     | 2     | 2     | 1     |

## Next question is:

#### `Keywords Associated with the RTX 4000 series graphics card. (Keywords)`

```postgresql
SELECT SUM(CAST(content ILIKE '%ample%' AS INTEGER))                                                   AS goodword_ample,
       SUM(CAST(content ILIKE '%high%' AS INTEGER))                                                    AS goodword_high,
       SUM(CAST(content ILIKE '%cool%' AS INTEGER))                                                    AS goodword_cool,
       SUM(CAST(content ILIKE '%top-notch%' OR content ILIKE '%top notch%' AS INTEGER))                AS goodword_top,
       SUM(CAST(content ILIKE '%seamless%' AS INTEGER))                                                AS goodword_seamless,
       SUM(CAST(content ILIKE '%amazing%' OR content ILIKE '%amaze%' AS INTEGER))                      AS goodword_amaze,
       SUM(CAST(content ILIKE '%beautiful%' OR content ILIKE '%beauty%' AS INTEGER))                   AS goodword_beauty,
       SUM(CAST(content ILIKE '%plentiful%' OR content ILIKE '%plenty%' AS INTEGER))                   AS goodword_plenty,
       SUM(CAST(content ILIKE '%efficient%' AS INTEGER))                                               AS goodword_efficient,
       SUM(CAST(content ILIKE '%excellent%' OR content ILIKE '%excellency%' OR content ILIKE
                                                                               '%excels%' AS INTEGER)) AS goodword_excellent,
       SUM(CAST(content ILIKE '%great%' OR content ILIKE '%greatness%' OR content ILIKE '%greatest%' AS
           INTEGER))                                                                                   AS goodword_great,
       SUM(CAST(content ILIKE '%innovative%' AS INTEGER))                                              AS goodword_innovative,
       SUM(CAST(content ILIKE '%impressive%' OR content ILIKE '%impress%' AS INTEGER))                 AS goodword_impress,
       SUM(CAST(content ILIKE '%solid%' AS INTEGER))                                                   AS goodword_solid,
       SUM(CAST(content ILIKE '%flawless%' AS INTEGER))                                                AS goodword_flawless,
       SUM(CAST(content ILIKE '%smooth%' AS INTEGER))                                                  AS goodword_smooth,
       SUM(CAST(content ILIKE '%fantastic%' AS INTEGER))                                               AS goodword_fantastic,
       SUM(CAST(content ILIKE '%outstanding%' AS INTEGER))                                             AS goodword_outstanding,
       SUM(CAST(content ILIKE '%good%' AS INTEGER))                                                    AS goodword_good
FROM processed_competitors; -- This should be changed to table name on different observations

```

| Keyword              | amount_for_competitors | amount_for_nvidia |
|----------------------|------------------------|-------------------|
| goodword_ample       | 4                      | 7                 |
| goodword_high        | 175                    | 255               |
| goodword_cool        | 81                     | 144               |
| goodword_top         | 0                      | 0                 |
| goodword_seamless    | 3                      | 7                 |
| goodword_amaze       | 23                     | 33                |
| goodword_beauty      | 18                     | 31                |
| goodword_plenty      | 5                      | 3                 |
| goodword_efficient   | 17                     | 45                |
| goodword_excellent   | 21                     | 8                 |
| goodword_great       | 54                     | 47                |
| goodword_innovative  | 3                      | 4                 |
| goodword_impress     | 25                     | 31                |
| goodword_solid       | 11                     | 21                |
| goodword_flawless    | 0                      | 0                 |
| goodword_smooth      | 8                      | 24                |
| goodword_fantastic   | 15                     | 7                 |
| goodword_outstanding | 4                      | 3                 |
| goodword_good        | 61                     | 69                |

### Table for badwords

```postgresql
SELECT SUM(CAST(content ILIKE '%insufficient%' OR content ILIKE '%insufficiency%' AS INTEGER))                AS badword_insufficient
     , SUM(CAST(content ILIKE '%low%' OR content ILIKE '%lower%' OR content ILIKE '%lowest%' AS INTEGER))     AS badword_low
     , SUM(CAST(content ILIKE '%overheating%' OR content ILIKE '%overheated%' AS INTEGER))                    AS badword_overheating
     , SUM(CAST(content ILIKE '%disappointing%' OR content ILIKE '%dissapointed%' AS INTEGER))                AS badword_disappointing
     , SUM(CAST(content ILIKE '%limited%' OR content ILIKE '%limiting%' AS INTEGER))                          AS badword_limited
     , SUM(CAST(content ILIKE '%poor%' OR content ILIKE '%poorer%' OR content ILIKE '%poorest%' AS INTEGER))  AS badword_poor
     , SUM(CAST(content ILIKE '%subpar%' AS INTEGER))                                                         AS badword_subpar
     , SUM(CAST(content ILIKE '%lacking%' OR content ILIKE '%lacked%' AS INTEGER))                            AS badword_lacking

     , SUM(CAST(content ILIKE '%power-hungry%' OR content ILIKE '%power hungry%' AS INTEGER))                 AS badword_power_hungry
     , SUM(CAST(content ILIKE '%terrible%' AS INTEGER))                                                       AS badword_terrible
     , SUM(CAST(content ILIKE '%underwhelming%' OR content ILIKE '%underwhelmed%' AS INTEGER))                AS badword_underwhelming
     , SUM(CAST(content ILIKE '%struggling%' OR content ILIKE '%struggled%' AS INTEGER))                      AS badword_struggling
     , SUM(CAST(content ILIKE '%suboptimal%' OR content ILIKE '%sub optimal%' OR content ILIKE
                                                                                 '%sub-optimal%' AS INTEGER)) AS badword_suboptimal
     , SUM(CAST(content ILIKE '%overhyped%' OR content ILIKE '%over hyped%' OR content ILIKE
                                                                               '%over-hyped%' AS INTEGER))    AS badword_overhyped
     , SUM(CAST(content ILIKE '%lacking%' OR content ILIKE '%lacked%' AS INTEGER))                            AS badword_lacking
     , SUM(CAST(content ILIKE '%bad%' AS INTEGER))                                                            AS badword_bad
FROM processed_competitors; -- This should be changed to table name on different observations

```

| Keyword               | amount_for_competitors | amount_for_nvidia |
|-----------------------|------------------------|-------------------|
| badword_insufficient  | 0                      | 0                 |
| badword_low           | 182                    | 318               |
| badword_overheating   | 1                      | 3                 |
| badword_disappointing | 2                      | 5                 |
| badword_limited       | 54                     | 47                |
| badword_poor          | 3                      | 3                 |
| badword_subpar        | 0                      | 0                 |
| badword_lacking       | 2                      | 1                 |
| badword_power_hungry  | 0                      | 0                 |
| badword_terrible      | 3                      | 1                 |
| badword_underwhelming | 1                      | 1                 |
| badword_struggling    | 2                      | 2                 |
| badword_suboptimal    | 0                      | 0                 |
| badword_overhyped     | 0                      | 0                 |
| badword_lacking       | 2                      | 1                 |
| badword_bad           | 10                     | 20                |