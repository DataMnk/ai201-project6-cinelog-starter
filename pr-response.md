# PR Response Doc — CineLog Watchlist Feature

## AI Usage
<!-- Fill in at the end — how you used AI tools during this project -->

## Comment 1 — Rename
**What I did:**
After reading the main scripts, particularly models.py, collection_service.py, 
and test_collection.py, I focused on understanding the functions and their 
differences by comparing them side to side. The function save_to_watchlist() in 
watchlist_service.py was similar to add_to_collection() in collection_service.py 
but the naming convention didn't match, so I renamed it to add_to_watchlist() to 
follow the verb_to_noun convention, which the reviewer's comment on the PR also 
confirmed directly.

**How I verified:**
After renaming the function in watchlist_service.py, I had to do a bunch of 
environment fixes on my side and reinstall the requirements. After that, Claude 
flagged that save_to_watchlist() was still being called inside add_film() in 
routes/watchlist.py, so I fixed that call site too. Then we ran the tests and 
all 4/4 passed.

## Comment 2 — Deduplication
**What I did:**
Since this case was already well handled for the collection case, I mirrored 
the pattern from add_to_collection() and applied it to add_to_watchlist(). 
I added a new exception called AlreadyInWatchlistError, along with the 
"if existing" check that queries for a matching user_id + film_id combination 
before creating the entry.

**How I verified:**
I ran pytest tests/test_collection.py -v again — all 4 tests passed, after 
fixing a couple of syntax typos. At this point I hadn't yet written a test that 
directly exercises the new AlreadyInWatchlistError check — that came later in 
Comment 3, with test_add_to_watchlist_duplicate_raises.

## Comment 3 — Missing test
**What I did:**
I created a new file, tests/test_watchlist.py, mirroring the structure of 
test_collection.py — same three fixtures (app, sample_user, sample_film) and 
the same testing pattern. I added 3 tests total: test_add_to_watchlist_creates_entry 
and test_add_to_watchlist_duplicate_raises weren't explicitly requested, but I added 
them to match the coverage style already used for collections. The one the reviewer 
actually asked for was test_add_to_watchlist_nonexistent_film_raises, which mirrors 
test_add_to_collection_nonexistent_film_raises from test_collection.py. One thing 
I had to think about: I used fake_film_id = 999999 (an integer) instead of a fake 
UUID, since Film.id is still db.Integer at this point in the project (before the 
Comment 6 rebase to UUIDs).

**How I verified:**
I ran pytest tests/test_watchlist.py -v first — all 3 passed. Then I ran the full 
suite with pytest tests/ -v to make sure nothing else broke — 7/7 passed 
(4 from test_collection.py + 3 from test_watchlist.py).

## Comment 4 — Default visibility
**My position:**
I'd change the default from public=True to public=False, so watchlists are 
private unless a user actively chooses to share them.

**Reasoning:**
A watchlist is different from a collection. CollectionEntry (films already watched 
and rated) doesn't even have a public field, it's always visible. WatchlistEntry 
represents films someone is just curious about, not something they've committed to 
or rated yet. For example, if I add "Devil Wears Prada 2" to my watchlist, it's not 
really about privacy from other people specifically, it's that the more data you put 
out publicly, the easier it is for platforms to build a detailed profile on you. 
I'd rather default to staying anonymous and low-profile, and only go public if I 
actually choose to.

**Tradeoff acknowledged:**
The real cost here is that CineLog's social discovery feature (seeing what friends 
want to watch) gets used less if watchlists default to private, since most people 
never bother changing default settings. I'm okay with that tradeoff. I can just tell 
friends in person what I want to watch. I'd rather CineLog lean toward the kind of 
privacy-by-default, humane-tech-style design that groups like the Center for Humane 
Technology advocate for, instead of optimizing defaults for engagement or social 
exposure.

## Comment 5 — Sort order

**My position:**
I agree with the reviewer. I'll implement date-added order (most recent first) 
as the default, matching the reviewer's suggestion.
**Reasoning:**
My first instinct was actually to sort by genre, since I don't want horror or 
other inappropriate content showing up front when I open my watchlist. But the 
genre field in the Film model is a free-text string, not a fixed set of 
categories, so "Horror", "horror", and "Horror/Thriller" would all sort as 
different values. That would make genre sorting inconsistent with the data as 
it exists right now. Given that, date-added order makes more sense as the 
default. It matches what get_collection() already does, and most users probably 
do want to see what they added most recently, like the reviewer said.

**Engagement with reviewer's point:**
I agree that most users want to see recently added films first, that's a 
reasonable default for a watchlist that's meant to be acted on soon, not 
browsed like a catalog. I implemented date-added order as requested.

## Comment 6 — Rebase
**What conflicted:**
**How I resolved it:**
**How I verified no conflict remains:**

## PR Description
<!-- Written at the end — feature overview, design decisions, manual testing steps -->