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


