# Movie Similarity Recommender (Cosine Similarity with MapReduce)

## 📌 Overview

This project implements a **movie recommendation engine** using **cosine similarity** on user ratings. Built with the **MapReduce paradigm using Python's `mrjob`**, the system identifies and ranks similar movies based on co-occurrence and rating similarity.

## 🛠 Technologies Used

- Python
- [mrjob](https://mrjob.readthedocs.io/en/latest/)
- Hadoop or local runner
- MovieLens 100k dataset (`u.data`, `u.item`)

## 📂 Input Data

- `u.data` — Contains user ratings in the format: `userID \t movieID \t rating \t timestamp`
- `u.item` — Metadata file containing movieID and movie name, delimited by `|`

## 🔍 Features

- Calculates **cosine similarity** between movie pairs.
- Filters out weak matches (low score or few co-ratings).
- Outputs top similar movie pairs with:
  - Similarity score
  - Number of co-ratings
- Uses **combinations** to find all pairs each user has rated.
- Applies **Facade pattern** to simplify the computation stages.

## 🧠 How It Works

### MapReduce Steps:

1. **Step 1: Parse Input**
   - Mapper: Emit `(userID, (movieID, rating))`
   - Reducer: Group ratings by user

2. **Step 2: Generate Movie Pairs**
   - Mapper: Emit `((movieID1, movieID2), (rating1, rating2))`
   - Reducer: Compute cosine similarity and count co-ratings

3. **Step 3: Sort and Output**
   - Mapper: Reformat for sorting by movie name and score
   - Reducer: Output final similar movie pairs

## 🧪 Example Output

"Star Wars (1977)" ("Empire Strikes Back, The (1980)", 0.97, 55)
"Toy Story (1995)" ("Bug's Life, A (1998)", 0.96, 48)
