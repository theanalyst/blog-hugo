+++
title = "2014: the year in books, using Goodreads, pandas & Hy"
date = "2014-12-21"
tags =["Hy", "Goodreads", "Pandas", "Kindle"]
comments = true
image = "images/kindle.jpg"
+++
``` Note: The content has been updated to reflect the new kwargs style
in Hy, as such the snippets only work with the current Hy master ```

As 2014 is coming to a close, I thought it would be a nice time to
review the year as far as reading was concerned. Since I track the
books I read using Goodreads, I felt it would be an interesting
experiment to see some numbers from Goodreads. Though Goodreads
provides an api, it doesn't seem to be directly useful to get some
numbers from a user account (I could be wrong here). Fortunately, an
export to csv option is provieded in the account which helps in our
favor.

For any kind of data analysis in python, [pandas][1] is a brilliant
library. Also it does most of the heavy lifting as far as processing
csv files are concerned. Since [Hy][2] works wherever python works, I
thought it would be an interesting experiment to use Hy to parse the
data.

To start with reading csv is a simple call to pandas' `read_csv`
function. This can be done with only the interesting fields. 

    :::hylang
	(import pandas
        [numpy :as np]
        [matplotlib.pyplot :as plt]
        [seaborn :as sns])

    (defn parse-goodreads-csv [filepath]
	  (let [[required-fields ["Title" "Date Read" "Bookshelves"
                              "Number of Pages" "Original Publication Year"]]
        (pandas.read_csv filepath :usecols required-fields
			             :index-col "Date Read" :parse-dates true)))


Now taking out only the column's we're interested in & filtering out
the data from only a particular year can be done by

	:::hylang
    (defn books-in-year [dataframe year]
	  (let [[day1 (fn [y] (+ (str y) "-01-01"))]]
        (slice (. dataframe ix) (day1 year) (day1 (inc year)))))

`ix` returns the index of dataframe, since we'll be indexing by date
read, this allows us to select the required range, by simply selecting
a date range from the beginning of the year to the next year.

Since it would be interesting to see the count of books as well as the
pages on a monthly basis, we should filter the data on a monthly
basis. Pandas offers a [`groupby`][3] & `aggregate` much similiar to
SQL like queries. So all we have to do is to group pages by month
here.

    :::hylang
	(defn aggregate-by-month [dataframe params]
	  "Group a particular key by month"
      (-> (.groupby dataframe (. dataframe index month))
	      (.aggregate params)))

The above function assumes that the dataframe object is already
indexed by a timeframe data, (Date Read in our case), then a simple
groupby month is performed and supplied to `aggregate` which does a
cumulative operation on the grouped data according the supplied
functions. The threading operator `->` makes the result of the first
call as the first argument of the function. Now we have enough to get
some stats for the year.

    :::hylang
	(defn process [filepath]
	  (let [[books-in-2014
              (-> (parse-goodreads-csv filepath)
                  (books-in-year 2014))]
            [pages-per-month (-> (. books-in-2014 [["Number of Pages"]])
                             (aggregate-by-month ["sum" "count" np.mean]))]]
      (print "Pages read in 2014 "
             ((. books-in-2014 ["Number of Pages"] sum)))
      (print "Pages read in kindle"
             ((. books-in-2014 [(= books-in-2014.Bookshelves "kindle")]
                 ["Number of Pages"] sum)))
      (print "Monthly Stats")
      (print pages-per-month)


First, the `parse-csv` function is given only the interesting columns,
also indexing is done via date read, we get `pages-per-month` via the
aggregate & groupby function shown earlier. Counting the pages read in
Kindle was done by summing columns containing the Book shelves
(alternatively if you had marked the editions correctly another field
suggest the Kindle edition..).

    :::txt
	Pages read in 2014  6659.0
	Pages read in kindle 4489.0
	Monthly Stats
    Number of Pages
                sum  count        mean
     1              1061      3  353.666667
     2               136      2   68.000000
     6               373      1  373.000000
     8               484      2  242.000000
     9              1445      8  180.625000
     10             2126     17  125.058824
     11              262      1  262.000000
     12              772      6  128.666667


Finally this is the plot of pages

![pages_per_month](/images/pages-per-month.png)

Overall 2014 was a good year as far as reading was concerned, read a
lot of classics which had been on my backlog for quite sometime. Also
read *The Guide*, which was pending for quite some time. Next year I
should follow a more consistent schedule :), there were months with no
reading at all and a few with too much (the peak in october was
probably due to the combined holidays here, at that time) The raw data
and output (as well as the program) are available in my [github][4]
repo.

PS For those of you interested in knowing what I've been reading
checkout [my goodreads 2014 list][5]. Do suggest anything worth
reading in the comments. Cya!

[1]: http://pandas.pydata.org
[2]: http://hylang.org
[3]: http://pandas.pydata.org/pandas-docs/stable/groupby.html
[4]: http://github.com/theanalyst/cuddlebear
[5]: https://www.goodreads.com/review/list/14029645-abhishek?read_at=2014&view=covers
