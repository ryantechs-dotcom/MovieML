# Item-Item Movie Recommender with MapReduce

Finds similar movies from user ratings using item-item collaborative filtering. Cosine similarity is computed across co-rated movie pairs in a three-stage MapReduce job written with `mrjob`. The same job runs on a laptop or scales out unchanged to Hadoop or Amazon EMR.

![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![mrjob](https://img.shields.io/badge/mrjob-MapReduce-66CCFF?style=flat-square)
![Hadoop](https://img.shields.io/badge/Hadoop%20%2F%20EMR-ready-66CCFF?style=flat-square&logo=apachehadoop&logoColor=black)

## How it works

| Step | Mapper | Reducer |
|---|---|---|
| 1. Group by user | `line → (userID, (movieID, rating))` | collect each user's `(movie, rating)` list |
| 2. Build pairs | for every pair of movies a user rated, emit `((m1, m2), (r1, r2))` in both orders | cosine similarity + co-rating count |
| 3. Rank | re-key as `((movie name, score), (similar movie, n))` so the shuffle sorts results | emit `movie → (similar movie, score, n)` |

**Quality filter.** A pair is kept only if more than 10 users rated both movies and cosine similarity exceeds 0.95. This removes the high-similarity, low-evidence pairs that dominate raw output.

**Scaling note.** Step 2 is O(k²) in the number of movies each user rated, so heavy raters dominate the shuffle. Capping or sampling per-user histories is the usual fix at larger scale.

## Output format

```
"Movie A (year)"   ["Movie B (year)", <cosine score>, <co-rating count>]
```

## Run it

The job reads the **MovieLens 100K** format (`u.data` tab-separated, `u.item` pipe-separated), available from [GroupLens](https://grouplens.org/datasets/movielens/100k/).

```bash
pip install mrjob
python MovieSimilarities.py --items=ml-100k/u.item ml-100k/u.data > similarities.txt

# on EMR (requires AWS credentials configured for mrjob)
python MovieSimilarities.py -r emr --items=ml-100k/u.item ml-100k/u.data
```

## Next steps

- Add a parser for the included MovieLens 1M files (`::`-delimited `ratings.dat` / `movies.dat`)
- Mean-center ratings (adjusted cosine) to correct for users who rate everything high
- Serve top-N neighbors per movie from the output as a lightweight "because you watched" API
