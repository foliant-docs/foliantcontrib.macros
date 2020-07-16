[![](https://img.shields.io/pypi/v/foliantcontrib.macros.svg)](https://pypi.org/project/foliantcontrib.macros/)  [![](https://img.shields.io/github/v/tag/foliant-docs/foliantcontrib.macros.svg?label=GitHub)](https://github.com/foliant-docs/foliantcontrib.macros)

# Macros for Foliant

*Macro* is a string with placeholders that is replaced with predefined content during documentation build. Macros are defined in the config.


## Installation

```shell
$ pip install foliantcontrib.macros
```


## Config

Enable the preprocessor by adding it to `preprocessors` and listing your macros in `macros` dictionary:

```yaml
preprocessors:
  - macros:
      macros:
        foo: This is a macro definition.
        bar: "This is macro with a parameter: {param}"
```


## Usage

Here's the simplest usecase for macros:

```yaml
preprocessors:
  - macros:
      macros:
        support_number: "8 800 123-45-67"
```

Now, every time you need to insert your support phone number, you put a macro instead:

```html
Call you support team: <macro>support_number</macro>.

Here's the number again: <m>support_number</m>.
```

Macros support params. This simple feature may make your sources a lot tidier:

```yaml
preprocessors:
  - macros:
      macros:
        jira: "https://mycompany.jira.server.us/jira/ticket?ID={ticket_id}"
```

Now you don't need to remember the address of your Jira server if you want to reference a ticket:

```html
Link to jira ticket: <macro ticket_id="DOC-123">jira</macro>
```

## Realworld example

You can combine Macros with tags by other Foliant preprocessors.

This can useful in documentation that should be built into multiple targets, e.g. site and pdf, when the same thing is done differently in one target than in the other.

For example, to reference a page in MkDocs, you just put the Markdown file in the link:

```html
Here is [another page](another_page.md).
```

But when building documents with Pandoc all sources are flattened into a single Markdown, so you refer to different parts of the document by anchor links:

```html
Here is [another page](#another_page).
```

This can be implemented using the [Flags](https://foliant-docs.github.io/docs/preprocessors/flags/) preprocessor and its `<if></if>` tag:

```html
Here is [another page](<if backends="pandoc">#another_page</if><if backends="mkdocs">another_page.md</if>).
```

This bulky construct quickly gets old when you use many cross-references in your documentation.

To make your sources cleaner, move this construct to the config as a reusable macro:

```yaml
preprocessors:
  - macros:
      macros:
        ref: <if backends="pandoc">{pandoc}</if><if backends="mkdocs">{mkdocs}</if>
  - flags
```

And use it in the source:

```html
Here is [another page](<macro pandoc="#another_page" mkdocs="another_page.md">ref</macro>).
```

> Just remember, that in this use case `macros` preprocessor must go *before* `flags` preprocessor in the config. This way macros will be already resolved at the time `flags` starts working.
