# dynamic-klipse-update
Exploring how to get web interfaces using `klipse` to update dynamically

This is a total hack as I'm banging around trying to figure out how to
get page content to update dynamically while doing lengthy computations
in ClojureScript using `klipse`.

To play with this clone the repo and run `lein ring server`. After that
has downloaded all the stuff then you should be able to go to
http://localhost:3000 and see the static website created by Cryogen. The page
I've been playing with is in `content/md/pages/another-page.md` and should
be visible in the browser at http://localhost:3000/blog/pages-output/another-page/

My idea is that computing the first 40 fibonacci numbers should take enough
time that it would make sense to see the web page updated after each one
instead of having to wait for them _all_ to be computed before anything
is displayed.
