# Jargon-free explanation of the hashing trick

So the hashing trick is something that gets mentioned in ML papers a
lot, but with the unfortunate caveat that every paper I've seen
mentions it in passing and assumes you know what it is and why you'd
want to use it. This is my attempt to explain that so that you too, in
your future papers can confidently name-drop your usage of the hashing
trick.

So the basic issue is that most methods of representing features rely
on something called one-hot encoding (in a nut-shell, this means that
you have to binarize everything. So for example to store the word
'hello' you'd instead store a really long row that is full of 0s for
every word in the English language that isn't "hello" and one 1 under
the "hello" column.

While this is a nice data structure for computers to learn from, it's
not very useful for humans.

// TODO: merge above and below intros

The usual way is to do what the ML folks call one-hot encoding. This is
basically just a fancy way of saying make two columns and mark them as
on and off.  One of the most common ways this is used is in what gets
called the "bag-of-words" model. Imagine taking a dictionary and an
excel spreadsheet, and entering each word in the dictionary as a
column in your spreadsheet (bag-of-words doesn't bother with the
definitions, just the words themselves).

Now let's say you've got a short paragraph of text and you want to use
your new dictionary spreadsheet to keep track of how many times words
show up in that paragraph. Given this paragraph:

> Anononick is an awesome writer, but he doesn't jump over dogs very
  much. Then again, outside of typing exercises neither do foxes,
  regardless of how quick they are. On the plus side though, very few
  brown foxes are better writers than Anononick.


The easiest way to do that is to look through your excel spreadsheet
and find the column for each word, and then in the next row enter the
number of times that word occurred. So if you were starting with the
first sentence you'd immediately run into a problem: Anononick is my name,
and isn't in the dictionary. OK fair enough, let's put a new column on
the end with my name on it, and put a 1 under it. At this point your
Excel sheet might look like this:


| zymurgy | zythum | Anononick |
----------|--------|-----|
|         |        | 1   |

By the time you get to the end of the first sentence it'll be more
like this:


<!-- 
	TODO: update table to show that order doesn't matter, and ideally
	use a repeating word so that the values aren't all 1
-->
 
 // TODO: alphabetize the order of words in the table and use `...` to show 
 // that this is not just representing the sentence but the whole dictionary. Ideally also
 // use a sentence with a repeating word for the example so that the values aren't all 1
 
Anononick | is | an | awesome | writer | but | he | doesn't | jump | over | dogs | very | much
----|----|----|---------|--------|-----|----|---------|------|------|------|------|-----
 1  |  1 | 1  |    1    |    1   |  1  |  1 |   1     |   1  |   1  |  1   |  1   |   1 
 

(by the way, you may have noticed that this approach doesn't
store the order of the words. This is one of the principal drawbacks
of the bag of words model, but that's a topic for another post)

So the biggest problem here is that we had to create a massive
spreadsheet with every word in the dictionary in it just to represent
a single paragraph. At this point you may be wondering why in the
world you would want to do this in the first place. Well, the 
real answer is that you don't, this is a horrible way for humans to
look at words. But this format does make it easy for machine learning
(and other) algorithithms to work with the data

Anyway, that was all background. On to the main point: hashing.

So hashing basically means doing complicated math on something so that the same input to the hash function
always gives you the same output. 

<!--
Use a good metaphor. Fingerprints, social security numbers, something like this. Main point being
that a hash function will always return the same output for the same input. If I take your fingerprint
that fingerprint will always be unique to you even though it's not you.
-->


So the hashing trick is basically just to take these inputs (in this case words) and map them to
a consistent set of outputs.

You can parameterize your hash function with what I'll call a resolution as well: let's say I want my hash function to only
give me 5 possible outputs. 

     hash         |   output
------------------|---------
hash(Anononick)         |    3
hash(is)          |    4
hash(an)          |    1
hash(awesome)     |    2
hash(writer)      |    4
hash(but)         |    3
hash(he)          |    4
hash(doesn't)     |    4
hash(jump)        |    3
hash(over)        |    5
hash(dogs)        |    1
hash(very)        |    2
hash(much)        |    4



Why would I want to do that? Well this means that now I don't have to keep a column for every single
word in the dictionary, in fact I only need 5 columns. 

The downside of this is that now I have to worry about collisions:
`hash(Anononick)` and `hash(but)` are hashing different words, but they both hash to 3. This can be bad sometimes, but it turns out that most
of the time all this extra precision doesn't actually help the machine
learning algorightms do a better job, so the space and efficiency
savings are often worth the loss of precision.

Here's how the hashed table representation of the first sentence would
look (original words are annotated in the first row):


1:  dogs, an | 2:  very, awesome | 3:  jump, but, Anononick | 4:  much, doesn't, he, writer, is | 5:  over 
-------------|-------------------|--------------------|-----------------------------------|----------
2            |  2                |   3                |              5                    |    1


As you can see, we now have a much smaller table, but we have some
collisions. Using a hash function that was set to return numbers up to
10 (or even 100 or 1000 or more) would allow us to make collisions
unlikely but still have way less columns than a full dictionary.

A "feature vector" (at least in a bag of words representation) would
actually just be a row in a spreadsheet like the above.

This has a lot of implications. The most important of them being that
doing things this way makes things massively more predictable. We can
now be 100% sure what the size of our "feature vector" will be
(it'll be whatever size we tell our hash function to output!)

#### References I used when putting this together (in no particular order)

- [This quora answer](https://www.quora.com/Can-you-explain-feature-hashing-in-an-easily-understandable-way)
- [This archived writeup](https://web.archive.org/web/20160306094110/http://metaoptimize.com/qa/questions/6943/what-is-the-hashing-trick) that gets referenced everywhere but whose [original link](http://metaoptimize.com/qa/questions/6943/what-is-the-hashing-trick) appears to be down. 
- http://wikivisually.com/wiki/Hashing_trick
- The papers that inspired me to go try to learn this
	- [Feature Hashing for Large Scale Multitask Learning](https://arxiv.org/pdf/0902.2206.pdf)
	- [Hash Kernels for Structured Data](http://www.jmlr.org/papers/volume10/shi09a/shi09a.pdf)

