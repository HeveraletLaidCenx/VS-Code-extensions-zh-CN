# Writing a TextMate Grammar: Some Lessons Learned

### February 17, 2014 at 14:35:46

Although TextMate has been around for a long time (in computer years) and many language bundles exist, it is startling to find that the process of writing a language grammar remains poorly documented. Having recently managed to write a [grammar of my own](https://github.com/mattneub/AsciiDoc-TextMate-2) for the first time, here are some things I learned along the way.

## Time Involved

Writing a grammar can be a slow business. It took me some weeks just to prepare, collecting information and locating and studying the existing instructions and documentation.

It then took me about three weeks of extremely frustrating, difficult work before I got my *first two scopes* working in the grammar. (This was mostly because of undocumented things *you* mustn’t do and things that TextMate *can’t* do, both of which I tell you about later on this page.) After that, though, it was remarkably smooth sailing, and I was able to finish the entire grammar in a week. Here are some git log excerpts:

```
Date:   Wed Jan 15 13:59:57 2014 -0800
    initial commit

Date:   Wed Feb 5 09:20:18 2014 -0800
    finally got recursively related match patterns working!

Date:   Wed Feb 12 19:58:51 2014 -0800
    ready for first release!
```

## How Scopes Are Styled

One of the chief reasons for writing a grammar is syntax coloring — which, like so much else about a grammar, depends on scopes. The rules for how a scope should be colored and styled, however, are to be found, not in the grammar itself, but in *themes* and also in *settings*.

In particular:

- In TextMate 2, the standard themes are all in the Themes bundle. The user will probably have selected one of these themes. You have no control over which it is; yet upon this choice depends how your grammar will affect the look of a document.
- Themes can also be hidden away in other bundles. (Fortunately, at the moment, no standard bundle that I use, other than the Themes bundle, contains any themes.)
- Settings can color and style scopes, like miniature one-rule themes.

Since themes and settings are going to have an effect on how your scopes are displayed, you should take some time to examine the existing themes, as well as looking through all settings of all bundles, to get some notion of what this effect might be. To save you some time, I will now summarize the information that I gathered in this regard.

### Standard Themes

Unfortunately, different standard themes affect different scopes; ideally you should note down *every* scope listed in *every* theme, but this would take too long. Here are some of the *main general scopes* that are styled by the standard themes; these are scopes that you generally *want* to use, because you can rely on the user to have selected a standard theme that will style them for you:

```
comment
constant
constant.character.escape
constant.language
constant.numeric
declaration.section entity.name.section
declaration.tag
deco.folding
entity.name.function
entity.name.tag
entity.name.type
entity.other.attribute-name
entity.other.inherited-class
invalid
invalid.deprecated.trailing-whitespace
keyword
keyword.control.import
keyword.operator.js
markup.heading
markup.list
markup.quote
meta.embedded
meta.preprocessor
meta.section entity.name.section
meta.tag
storage
storage.type.method
string
string source
string.unquoted
support.class
support.constant
support.function
support.type
support.variable
text source
variable
variable.language
variable.other
variable.parameter
```

### Settings

I was stunned to discover that, in addition to themes, a bundle’s settings can style scopes. It turns out that this fact is extremely important. For example, I scratched my head for about two weeks, trying to figure out why my italic text was *shown* as italic. I couldn’t find any theme that was doing this. Then one day I stumbled on the Text bundle, which contains the responsible setting.

Here are the chief bundle settings to be aware of:

- **Text bundle**: `markup.bold`, `markup.italic`, `markup.underline`. (The underlining imposed by `markup.underline` is also how the Hyperlink Helper bundle injects underlining onto your URLs.) There are some other settings here, but they affect things like soft wrapping (also something to watch out for).
- **Themes bundle**: `markup.heading.n`, `markup.quote`, `markup.raw.block`. These settings affect both the font and size of these scopes. In my opinion, they are among the most horrible side effects of using TextMate 2. (If I wanted font and size to vary through my document, I’d be using Microsoft Word, for heaven’s sake. I’m here for the *text*. It’s called *TextMate*, after all.) To avoid having your text affected by these settings, *avoid these scopes*. I have also disabled these settings in my copy of TextMate, but you can’t rely on every user to do that.
- **TextMate bundle**: `meta.separator`.

If you use or enable other bundles, keep your eyes peeled for settings that may perform scope styling. For example:

- **Diff bundle**: `markup.changed`, `markup.deleted`, `markup.inserted`; `meta.diff.header`, `meta.diff.index`, `meta.diff.range`.

The Source bundle doesn’t contain any style settings, but it does have some settings that determine what counts as a word character for purposes of code completion (as explained in [this article](http://blog.macromates.com/2012/clever-completion/)) as well as word selection and navigation, so you might want to be careful with the distinctions being made here.

## Standard Scopes

The official list of scopes that are considered standard or conventional is not quite the same as the list of scopes that are commonly styled by themes. It’s good to have this list on hand. The list as given in the [documentation](http://manual.macromates.com/en/language_grammars#naming_conventions) is as follows. I have starred once the entries that are present also in the list of commonly styled scopes (though note that for some reason there are commonly styled scopes that are not present in this list); I have double-starred the entries that are styled by settings:

```
comment*
    line
        double-slash
        double-dash
        number-sign
        percentage
        [character]
    block
        documentation
constant*
    numeric*
    character
        escape*
    language*
    other
entity
    name
        function*
        type*
        tag()
        section*
    other
        inherited-class*
        attribute-name*
invalid*
    illegal
    deprecated
keyword*
    control [control.import*]
    operator [operator.js*]
    other
markup
    underline**
        link
    bold**
    heading*
    italic**
    list*
        numbered
        unnumbered
    quote* (and **)
    raw
    other
meta
storage*
    type [type.method*]
    modifier
string* (and string source*)
    quoted
        single
        double
        triple
        other
    unquoted*
    interpolated
    regexp
    other
support
    function*
    class*
    type*
    constant*
    variable*
    other [other.variable*]
variable
    parameter*
    language*
    other*
```

The Bundle Development bundle will also assist you as you edit your grammar by marking unusual scope names.

Note that although the documentation claims that `meta` scope is not styled, nevertheless some `meta` subscopes are in the list of scopes styled by themes and settings above.

## Regular Expressions

Grammar match rules are written as Ruby regular expressions. You need to be conversant with these! Download a copy of the [Oniguruma regular expression syntax](http://www.geocities.jp/kosako3/oniguruma/doc/RE.txt) (used by Ruby) and keep it handy. Things to note:

- Greedy, reluctant, and possessive matches
- Lookbehind and lookahead — your match rules are likely to make *extensive* use of these
- Shy groups, named groups, named group references, and “variables” (the “Tanaka Akira Special”, allowing you to refer *forward* to a group by number or name) — I found named groups, in particular, extremely helpful for clarifying complicated expressions
- `\G` — This is a way of anchoring an attempted match in exactly the place where the previous match ended (useful when a grammar works by carving up a document *completely* into contiguous scopes)

It will also be a good idea to keep [Rubular](http://www.rubular.com/) open in your browser. It allows you to test regular expressions, indicating matched groups.

Bear in mind, however, that because of the way the TextMate parser surveys your document, all regular expressions used in grammar match rules must apply to a *single line at a time*. A single expression cannot embrace multiple lines. Thus it is possible to write a regular expression that appears to work in Rubular (or TextMate’s regex Find) but will fail as part of a grammar.

Another pitfall is that TextMate will not complain if you write a bad regular expression; the expression will just fail silently. That’s probably a good thing in general, but it can drive you mad while trying to understand why your regular expression isn’t working. This is another reason why Rubular is useful; it will show a descriptive error message if the expression itself is syntactically faulty.

Regular expressions will be surrounded by single quotes in the grammar. To express a single quote within a regular expression in a grammar, use two single quotes in succession.

## Grammar Structure

We now come to the actual apparent structure of a language grammar. I say “apparent” because I’m just guessing, based on my experience. The documentation is shockingly uninformative about this. There is no formal complete definitive specification, so far as I have discovered, of the structure of a grammar. Moreover, many aspects of a grammar are described in separate and scattered documents that can be difficult to discover. You should certainly keep on hand a copy of the [grammar page in the manual](http://manual.macromates.com/en/language_grammars#language_grammars), as well as James Edward Gray II’s [book](http://pragprog.com/book/textmate/textmate). But I will attempt to do a better job here of assembling and summarizing the facts.

### Property List Syntax

The grammar is portrayed by the bundle editor as an old-style property list. Under the hood, however, it is a new-style property list, i.e. XML. This means that every time you save, the grammar is “compiled” into a new-style property list, with an error dialog if you’ve made a mistake such that this can’t be done; and every time you abandon the grammar window, when you return to it, the window is repopulated by “decompiling” the saved XML into an old-style property list. Therefore:

- The grammar contents may not look the same when you return to it as when you left it; for example, things whose order does not matter (dictionary elements) may appear in a different order.

- If you leave the grammar window after seeing the error dialog and without correcting the error, *you will lose your recent work*, because your recent work wasn’t saved, and now it has been wiped out by reading and decompiling the previously saved XML from disk.

  (This is more of a problem than you might expect, because the bundle editor window is responsive to clicks when it is not frontmost. Thus, you might switch to another window to study something about why your property list might be invalid, then click on the bundle editor window to switch back, and discover that, because of *where* you clicked, you’ve accidentally left your grammar window and have lost all your recent work. I know this from experience, obviously! My solution is: before switching away from the bundle editor with unsaved changes, copy its contents and paste them into BBEdit for safekeeping.)

An old style property list has two kinds of collection: arrays and dictionaries.

- An array, bounded by parentheses `()`, is an *ordered* list of elements separated by *comma*. It does no harm to follow the last element with a comma as well, and I recommend that you do so.

- A dictionary, bounded by curly braces `{}`, is an *unordered* list of name–value pairs separated by and ending with *semicolon*. A name–value pair is joined by an equal sign. The name does not have to be quoted, and usually will not be, unless it contains spaces or numbers. String values will be single quoted.

  **NOTE**: In the grammar structure, *any* dictionary can contain a `comment` entry. This can be extremely useful and I recommend liberal use of this feature.

### Top Level Structure

Before I can describe the top level of a language grammar, I need to tell you that a *match rule* (also called a *pattern*) is a dictionary. But I’m not going to tell you what’s *in* a match rule until later.

The top level, in the TextMate 2 bundle editor window, is a dictionary — that is, it’s a pair of curly braces `{}`. Some of the entries in the real underlying dictionary are now edited through fields in the bundle editor drawer. Of these, the most important is the grammar’s scope, listed in the drawer’s Grammar field. This is the main scope that will be assigned to the entire document. It is typically a specialization of some existing scope, in order to acquire related bundle-based features, though not usually in such a way that an existing grammar will be injected automatically on top of yours.

For example, `text.html.markdown` is not magically imposed upon by the Text bundle’s grammar (which is scoped to `text.plain`) or by the HTML bundle’s grammar (which is scoped to `text.html.basic`). But because it is a specialization of `text.html`, it does magically acquire any features scoped to `text.html`, such as the Command-Ampersand keyboard shortcut from the HTML bundle.

On the other hand, in the Bundle Development bundle, the Language Grammar grammar is scoped as `source.plist.textmate.grammar` exactly so that the Property List bundle’s Property List (Old-Style) grammar (`source.plist`) *can* be magically imposed upon it; the former is a pure specialization of the latter.

From here on, let’s confine ourselves to what’s shown in the bundle editor window itself. The top-level dictionary can contain (aside from a comment) a `patterns` array and (optionally) a `repository` dictionary, and that’s all.

- A `patterns` array is an array of match rules. The fact that it’s an array is important, because *order matters*. Matches are performed *in the order listed*. TextMate considers lines of a document one at a time, looking within each one for matches. When we make a match, that’s the end for everything in that line up to that point. Thus, you can effectively make conditional rules by judicious ordering of a `patterns` array.

- A repository is a dictionary, and it may contain two kinds of thing:

  - A repository may contain named *match rules*. The fact that the rules are *named* is important, because this means that a match rule can refer to another match rule *by name*, provided the latter is in a repository.

  - A repository may contain named *top levels*. That is, a repository entry might be a name paired with a dictionary containing a `patterns` array and (optionally) a `repository` dictionary.

    The `patterns` array inside a named dictionary inside a repository provides a way of naming (and thus referring to) a whole bunch of match rules simultaneously.

    The `repository` inside a named dictionary inside a repository has no structural significance — it is not special merely because it is at a deeper level — but it is certainly an organizational convenience, especially because the bundle editor gives you structural code folding.

To illustrate and explain, here’s the structure of the Markdown grammar:

```
{
  patterns = (... match rules ...);
  repository = {
    block = {
      patterns = (... match rules ...);
      repository = {
        blockquote = {... one match rule ...};
        ...
      };
    };
  };
}
```

With that structure, the name `block` can be used to refer to all of the `block` > `patterns` match rules simultaneously. The `blockquote` match rule, despite its depth (a repository within a repository), can be referred to from anywhere.

![image](https://www.apeth.com/nonblog/images/grammarTopLevel.png)

## Match Rules

A match rule, also known as a *pattern*, is where the power lies in a language grammar. It is the part of the grammar that actually instructs TextMate’s parser what to do: “as you walk through the text, look for this pattern and assign this scope.”

A match rule, as I’ve already said, is a dictionary. In describing the overall structure of the language grammar, I’ve already told you the two main places where match rules can go:

- A match rule can be one entry in a `patterns` array.
- A match rule can be a named dictionary in a repository.

I have not, however, said anything yet about what is *inside* a match rule dictionary. Now it’s time to do that. There are actually *three* possible structures for the contents of a match rule:

- An `include`.
- A `match` pattern.
- A `begin`/`end` (or `begin`/`while`) pattern.

We will consider each in turn. But first, I’ll mention two things that *any* match rule can contain:

- A comment. This is a name–value pair named `comment`.
- An off switch. This is a name–value pair named `disabled`, with value `1`. This is a convenient way to experiment or develop, leaving a rule in place while effectively commenting it out.

### An Include Match Rule

An `include` match rule is a way of saying: “Substitute for me the *named* match rule(s) I hereby specify.” Such a match rule contains, I believe, just one name–value pair. The name of that pair is `include`. The value is usually the name of something in the repository at any level, *preceded by a hash-sign*. (But it cannot, I believe, be `#repository`, even though that might be the name of something in the repository.)

To see include rules in action, let’s return to and fill out a little further the structure of the Markdown grammar:

```
{
  patterns = (
    { include = '#block'; }
  );
  repository = {
    block = {
      patterns = (
        { include = '#separator'; };
        { include = '#heading'; };
        { include = '#blockquote'; };
        ...
      );
      repository = {
        blockquote = {... one match rule ...};
        heading = {... one match rule ...};
        ...
        separator = {... one match rule ...};
      };
    };
  };
}
```

Consider how, by means of include rules, the `blockquote` match rule is actually brought into play. There is only one top-level match rule — the `include` rule specifying `#block`. This rule effectively substitutes for itself the entire array of `patterns` listed under `block` in the repository. These patterns are themselves all include rules. The result is exactly as if the top level `patterns` array *were* the include rule for `#separator`, then the include rule for `#heading`, then the include rule for `#blockquote`, and so on.

Those include rules, in turn, are virtually replaced by the rules that their values name — the *actual* `separator` rule from the repository, the *actual* `heading` rule from the respository, the *actual* `blockquote` rule from the repository, and so forth. Those actual rules are all defined in the second-level repository.

(Note that the order of the include rules in the `patterns` array is significant; an array is ordered. The order of the actual rules in the second-level repository, on the other hand, is undefined and unimportant; it happens to be shown as alphabetical, and that’s convenient, but it’s just a detail of the display implementation.)

The value of an `include` rule can alternatively be a *scope name*. This is a good way to inject an entire grammar inside yours. For example, in my AsciiDoc bundle, I assume that the contents of a passthrough block (never mind what that is) will be XML. Therefore, inside my match rule for a passthrough block, I have an `include` rule specifying `text.xml`. This causes the whole grammar from the XML bundle to come into play in this region of my document. In addition, TextMate 2 permits a single named item to be plucked from a foreign repository using the syntax `scopeName#itemName`.

The value of an `include` rule can also be `$self`, meaning the whole current grammar, or `$base`, the grammar inside which we are embedded. They are used by several prominent grammars (such as the Objective-C grammar), but I have never used them and I don’t quite understand the details.

As I mentioned at the outset, as far as I can tell, if a match rule contains an `include` key, that is its *only* key (except possibly for `comment` and `disabled`).

![image](https://www.apeth.com/nonblog/images/grammarMatchRuleType1.png)

### A One-Pattern Rule

A one-pattern rule must contain at least this key–value pair:

- The `match` key. Its value is a regular expression that the TextMate parser is to look for.

  This is, I am at pains to stress, a *single-line* regular expression. The TextMate parser will not examine more than one line of text at a time; thus, a regular expression that includes a newline (`\n`) anywhere other than as the last matched character will never match. By the same token, though, it can be quite useful (especially in a “prose” language grammar such as the Markdown grammar) to end a regular expression with `$\n?` as a way of snarfing up the newline at the end of the line if there is one.

A one-pattern rule may also contain this key–value pair:

- The `name` key. Its value is a scope, or an expression that evaluates to a scope. This is the scope that will be applied to the matched text. This scope assignment, in turn, has far-reaching implications for how the matched text will be styled by themes and settings, as I’ve already described above.

For example, here’s the rule from the Markdown grammar repository that picks out certain characters preceded by backslash:

```
escape = {
    name = 'constant.character.escape.markdown';
    match = '\\[-`*_#+.!(){}\[\]\\>]';
};
```

This means that any matches we find using the `escape` rule will be assigned the `constant.character.escape.markdown` scope. A theme or setting may then come along and style any `constant.character.escape` text.

The scope name can be, as I mentioned earlier, “an expression that evaluates to a scope”. What do I mean by *that*? Well, the scope name is actually what TextMate calls a *format string*. For more about these, see [this article](http://blog.macromates.com/2011/format-strings/). The syntax here is derived from shell programming syntax. With a format string, you can do things such as:

- Use a matched group as a term: Refer to it as `$0`, `$1`, and so forth.
- Use the value of a global variable as a term.
- Perform a transform, such as downcasing, on the value of a term.
- Do a find/replace on the value of a term.
- Supply a different result depending on whether a term (such as a matched group) is defined or not.

A one-pattern rule may also contain this key–value pair:

- The `captures` key. Its value is a dictionary. In this dictionary, each key is a number or name corresponding to a matched group from the `match` expression, and the corresponding value is a dictionary. This dictionary may contain one or both of these entries:
  - A `name`. This is the scope that will be assigned to the matched group text.
  - A `patterns` array. This is *a list of match rules* to be sought *within the matched group text* — thus permitting the search for matches to continue inside the matched text.

This use of the `captures` > `patterns` structure is extremely important. Without it, a stretch of text matched by a match rule is considered finished, and TextMate’s search proceeds to the rest of the line *after* the matched text. With it, the stretch of matched text itself becomes a candidate for further matches.

Here’s a simple minimal example. Imagine a toy markup language in which bold is delimited by asterisks and italic is delimited by underlines:

```
{ patterns = 
  (
    {
      name = 'markup.bold.toy';
      match = '\*.*?\*';
    },
    {
      name = 'markup.italic.toy';
      match = '_.*?_';
    },
  );
}
```

This works, but only for bold and italic stretches of text that are *separate*, like this:

```
*This is bold.* _This is italic._ 
```

An italic stretch cannot occur inside a bold stretch, because once a stretch of text has been matched as bold, it is discarded from further consideration. If we want italic inside bold, and vice versa, to be possible, we can use the `captures` > `patterns` structure to duplicate the italic pattern inside the bold pattern and the bold pattern inside the italic pattern:

```
{ patterns = 
  (
    {
      name = 'markup.bold.toy';
      match = '\*.*?\*';
      captures = {
        0 = { 
          patterns = (
            {
              name = 'markup.italic.toy';
              match = '_.*?_';
            }
          );
        };
      };
    },
    {
      name = 'markup.italic.toy';
      match = '_.*?_';
      captures = {
        0 = { 
          patterns = (
            {
              name = 'markup.bold.toy';
              match = '\*.*?\*';
            }
          );
        };
      };
    },
  );
}
```

That sort of thing is perfectly legal — the `patterns` array is a list of match rules, and we are providing match rules. And it works, correctly marking up text such as this:

```
*This is bold _containing italic_.* 
And _this is italic *containing bold*._
```

In this case, though, a neater approach would be to move the match rules into the repository so that they can be referred to by name, and use the `captures` > `patterns` structure *with the `patterns` match rules being `include` rules*, to continue the search for italic inside a matched bold stretch of text, and vice versa:

```
{ patterns = (
    { include = '#bold'; },
    { include = '#italic'; },
  );
  repository = {
    bold = {
      name = 'markup.bold.toy';
      match = '\*.*?\*';
      captures = { 0 = { patterns = ( { include = '#italic'; } ); }; };
    };
    italic = {
      name = 'markup.italic.toy';
      match = '_.*?_';
      captures = { 0 = { patterns = ( { include = '#bold'; } ); }; };
    };
  };
}
```

**WARNING:** Do *not* put a `patterns` array at the top level of a one-pattern match rule. It doesn’t generate any explicit error, but it doesn’t work correctly either. So, for example, *this is wrong*:

```
name = 'markup.bold.toy';
match = '\*.*?\*';
patterns = ( { include = '#italic'; } );
```

You are getting the benefit of my hard-fought experience here; it took me about a week of wrestling (and some direct help from Allan Odgaard) to learn the right way to do an `include` inside a one-pattern match rule.

![image](https://www.apeth.com/nonblog/images/grammarMatchRuleType2.png)

### A Two-Pattern Rule

A two-pattern rule must contain *two* regular expressions: either `begin` and `end`, or (new in TextMate 2) `begin` and `while`. These are still single-line regular expressions (the TextMate parser does not consider more than one line at a time), but there are two of them, so together they may delimit a stretch of text that embraces multiple lines.

- The `begin` key is a regular expression that will be sought initially.
- If the `begin` regular expression generates a match, TextMate will start looking immediately after it (starting in the same line, if the `begin` match did not snarf up the entire line) for the `end` or `while` expression.
  - If you provided an `end` expression, TextMate will keep walking the document until it matches the `end` expression, and will stop.
  - If you provided a `while` expression, TextMate will keep walking the document until it comes to a line where the `while` expression fails.

Neither of these pairs works quite the way I would have intuitively expected, so I’m going to say some more about them.

With `begin`/`end`, if the `end` pattern is *not* found, the overall match *does not fail*: rather, once the `begin` pattern is matched, the overall match runs to the `end` pattern *or to the end of the document*, whichever comes first. The underlying architectural reason is that the TextMate parser does not backtrack; once the `begin` pattern is matched, it is matched successfully and that’s that — TextMate can’t change its mind and decide that it shouldn’t have matched the `begin` pattern after all.

This is a major limitation in the degree of intelligence a language grammar can exhibit; it is said to be due to the need for speed, and perhaps for simplicity. In any case, it is intentional, and some grammars actually take deliberate advantage of this behavior. For example, the Property List Old-Style grammar starts like this:

```
{ patterns = (
    { begin = '(?=\(|\{)';
      end = '(?=not)possible';
```

The expression `(?=not)possible` is, uh, not possible; the next character can’t be “n” if the next character is “p”. This `end` pattern is designed to fail. The whole pair thus means: Once you have encountered a left parenthesis or a left curly brace, immediately snarf it up along with the entire remainder of the document (and just about everything else in the grammar then works through `include` rules inside that snarfed-up material).

With `begin`/`while`, things are even stranger. It’s rather difficult for me to deduce what the rules are; they are not yet documented, and I know of only one grammar that uses `begin`/`while`, namely the Markdown grammar, so my available examples are quite limited. Here’s what I’ve been able to guess:

1. Unlike `begin`/`end`, the text matched by `begin`/`while` stops at the end of the current line. Thus, if the `begin` is encountered and the `while` is never encountered, only *the rest of the line* containing the `begin` text is subsumed into the matched text.
2. After encountering a `begin` match, the TextMate parser looks in the *next line* to see if it can match the `while` pattern. If it can, it incorporates the matched text *and the entire rest of that line*, and looks in the *next* line to see if it can match the `while` pattern. This continues until a line is encountered in which the `while` pattern cannot be matched.

Again, there is no backtracking; failure to find the `while` pattern at all is not a reason for rejecting the `begin` match that was already found. Notice also that nothing about these rules says that the resulting matched stretches of text have to be contiguous. That’s because they *don’t* have to be contiguous! So, for example, let’s say that the `begin` pattern matches an asterisk (`*`) and the `while` pattern matches a plus sign (`+`). Then (matching stretches are shown in caps):

```
This is +no match.
This is *A MATCH.

This is +no match.
This is *A MATCH.
This is +ALSO A MATCH.
This is +A MATCH TOO.

This is +no match.
```

Well, I *told* you it was strange! However, it seems likely that I’m abusing the `begin`/`while` rule with this example. This curious “from here to the end of the line” match behavior is probably intended for patterns that mark an *entire* line in terms of *how it begins*. You’ll notice that *all* the Markdown grammar uses of the `begin`/`while` work that way. For example, the `blockquote` rule is written like this:

```
begin = '(^|\G)(>) ?';
while = '(^|\G)(>) ?';
```

That means, in essence: match an entire line that begins with one (`begin`) or more (`while`) greater-than signs (and perhaps one space after the greater-than sign), and keep matching entire *successive* lines (`while`) that begin that way.

Now let’s talk about other key–value pairs that can appear in a two-pattern rule.

- The `name` key. This is the scope (or an expression evaluating to a scope) to be applied to the *entire* matched stretch(es) of text starting at the start of the `begin` match.
- The `contentName` key. This is the scope to be applied to what’s *between* the `begin` match and the `end` match (or the end of the document). (With `begin`/`while`, this isn’t illegal, but it’s pointless, as it covers exactly the same stretch(es) as the `name` scope.)
- The `beginCaptures` and `endCaptures` (or `whileCaptures`) keys. These work like the `captures` key in the one-pattern rule: the value is a dictionary each of whose entries refers to a match group in the `begin`, `end`, or `while` pattern respectively, and is itself a dictionary containing `name` or `patterns` (or both). Thus you can apply a scope to, and/or continue searching inside, a matched stretch of text. If the name or number of the matched group happens to be the same for both the `begin` pattern and the other pattern, you can (if appropriate) use `captures` as a shorthand to avoid saying the same thing twice.
- The `patterns` key. This is like the `patterns` key inside the `captures` and `beginCaptures` (and so forth) dictionaries, but it applies to the region *between* the `begin` and `end` matches. Again, this is so that you can continue searching inside this stretch of text, which otherwise would be marked down as completed (and TextMate would start matching after the `end` match).

A surprise arises if a match specified in the `patterns` array can be satisfied by continuing beyond the `end` match. What happens is that the match *does* continue beyond the `end`, and causes the `end` to shift with it. For example, suppose we have this rule:

```
bold = {
  name = 'markup.bold.toy';
  begin = '\*';
  end = '\+';
  patterns = (
    { name = 'markup.italic.toy'; 
      match = '_.*?_';},
  );
};
```

If we start out like this, then things behave as we expect:

```
This is *a
bold+ stretch of text.
```

The words “a bold” are in bold, and that’s the end of that. But now:

```
This is *a
_bold+ stretch_ of text.
```

I would have expected that since the italic rule can’t be satisfied inside the matched bold stretch, it wouldn’t be matched. However, TextMate has loaded the *entire second line* for examination, and thus succeeds in matching the words “bold stretch” as italic. Moreover, we have now “punched through” the original `end` of the bold stretch, and the *entire rest of the line* becomes bold, along with the *entire rest of the document* until and unless we come to *another* bold `end` match (a plus sign). I find this both surprising and disappointing, as it greatly limits the kinds of natural logical structure you can successfully express.

(In addition, we are told that, in case both a subpattern (from the `patterns` array) and the main `end` pattern would end exactly on the same character, the `end` pattern wins unless you set the `applyEndPatternLast` key to `1`. I don’t understand what this means and I won’t try to explain it.)

![image](https://www.apeth.com/nonblog/images/grammarMatchRuleType3.png)

## Developing Your Grammar

In conclusion, some pearls of advice about the process of actual development of your grammar.

Collect all the documentation I’ve mentioned (including, I dare to suggest, this article) and leave everything open on your computer screen so that you can consult it as you work.

Now create your initial language grammar and start writing rules. Your first and constant question will be, at all times: “Is this working as I intend?” Thus you will need to have open, at all times, a TextMate document whose type is set to correspond to the grammar you are developing. As you create rules, add test text to the document so that you can see how it is affected.

The big issue, of course, is whether a given stretch of text is being assigned the scope you think it should be. Unfortunately, TextMate gives you no simple way to discover this. You cannot, for example, say “Show me all the scope runs of this document.” Nor can you search for a scope within a document. So you need some other way to examine the scope(s) in force at a point of the document.

One important technique is to select some text and then press Control-Shift-P or Control-Shift-Command-P. These are commands from the Bundle Development bundle that display the current scopes(s) in a tooltip (which, unfortunately, vanishes if you so much as twitch the mouse).

Another clever device is to impose your own theme containing a special artificial scope (such as `text.testing`) that it styles in a very prominent way; you can then temporarily use that special scope as the `name` or `contentName` value in a rule to see just where that rule is applied in your test document. A newly created theme is empty, so you’ll probably want to paste into your theme the contents of an existing theme. For example, copy the whole contents of the Mac Classic theme; make a new theme in your bundle; select the new theme’s contents and paste the Mac Classic theme contents in its place; and now add a scope such as this:

```
{   scope = 'text.testing';
    settings = {
        foreground = '#FFFFFF';
        background = '#0000FF';
    };
},
```

Now bring your test document to the front and choose View > Theme > YourTheme to apply your special theme to it.

The bad news here is that if you make a change to your theme, it is not registered immediately in your test document; you have to choose View > Theme > YourTheme *again* in order to make any changes take effect. (Like themes, certain other changes are applied only lazily. For example, if your bundle is to have Folding settings or Table of Contents settings, you’ll find that it can be quite difficult to nudge TextMate to reflect any changes you make in those settings.)

On the other hand, the good news — the *really* good news — is that any change you make in your *grammar* takes effect *as soon as you save*. Thus it is very easy to work simultaneously on the test document and the grammar. You can make a change in your grammar, save, and then immediately examine your test document to see how it is styled. If you’re using the special theme trick that I just suggested, you can create or modify a rule and set its scope to `text.testing` to see that it carves up the document the way you expect, before changing its scope to the real value you ultimately want it to have.

Finally, *be prepared to crash*. This happens, presumably, because the combination of the grammar and the test document has driven TextMate temporarily insane. However, that’s not bad; it’s good, because it’s better that you should crash TextMate *now*, while developing your grammar, than that TextMate should crash *later* and cause someone (maybe you) editing a document under the influence of your grammar to lose work on that document. Moreover, you probably will not lose any work on your grammar, because the evil change that you made in the grammar didn’t go into effect until you saved; so, *ipso facto*, you *did* save. (You might lose work on your test document, but that’s far less important. Still, you should be saving that test document often as well.)

To recover from such a crash, do *not* be a ninny (like me) and open the test document again in TextMate. That might merely cause TextMate to crash again! It is the combination of your grammar and *this test document* that is the source of the trouble. Instead, open TextMate without any document, get back to the bundle editor, and try to undo whatever bad thing you did that is causing the crash. It will not always be clear what this is, but I suspect that it is usually some sort of infinite recursion caused by a match rule containing an `include` rule containing itself.