# Dragonfly Data Science Blog

Based on Minimal Mistakes https://github.com/mmistakes/minimal-mistakes

## Code highlighting

We use:

- kramdown - the general markdown processor used by Jekyll.
- coderay - adds annotated classes to words in code blocks.

Then a css theme is used to make things pretty. These are named
`assets/css/coderay_*`. By default we use one that looks like github's code
highlighting.

An example:

~~~ python
def woohoo(text):
    print text
~~~

`_config.yml` defines the `default_lang` as Python. Which means you can get
away with:

~~~
def woohoo(text):
    print text
~~~

If you want another language, specify it:

~~~ haskell
main :: IO ()
main = do
    (style, refs) <- myinit

    hSetEncoding stdout utf8
    putStr "Hello"
~~~
