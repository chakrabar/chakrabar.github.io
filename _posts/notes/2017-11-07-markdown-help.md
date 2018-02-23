---
layout: post
title: "Markdown help"
excerpt: "Some quick Markdown examples"
date: 2017-11-07
categories: notes
share: true
comments: true
modified: 2017-11-07T22:11:53-04:00
---

## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/chakrabar/chakrabar.github.io/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

OR

* Bulleted
* List

1. Numbered
2. List
    1. Sub
    2. List
3. Continue

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)

```

//comment >> [comment]: # (This is one of the most common syntax for comments)

Code block within `` ```<code>``` ``. For more specific syntax highlighting, e.g. for C# use `` ```cs <code> ``` ``. See sample below

1. Simple code block highlight
```
public class CoolClass
{
    public string Name { get; set; }
    public int Id => 101;

    public CoolClass(string name, int id) => (Name, Id) = (name, id);
}
```

2. Code block with syntax highlighting
```cs
public class CoolClass
{
    public string Name { get; set; }
    public int Id => 101;

    public CoolClass(string name, int id) => (Name, Id) = (name, id);
}
```

### Table


| Employee         | Salary |                                                              |
|------------------|--------|--------------------------------------------------------------|
| [John Doe](#)    | $1     | Because that's all Steve Jobs needed for a salary.           |
| [Jane Doe](#)    | $100K  | For all the blogging she does.                               |
| [Fred Bloggs](#) | $100M  | Pictures are worth a thousand words, right? So Jane × 1,000. |
| [Jane Bloggs](#) | $100B  | With hair like that?! Enough said.                           |


| Header1 | Header2 | Header3 |
|:--------|:-------:|--------:|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|-----------------------------|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|=============================|
| Foot1   | Foot2   | Foot3   |

### Notices

**Watch out!** This paragraph of text has been [emphasized](#) with the `{: .notice}` class.
{: .notice}

**Watch out!** This paragraph of text has been [emphasized](#) with the `{: .notice--primary}` class.
{: .notice--primary}

**Watch out!** This paragraph of text has been [emphasized](#) with the `{: .notice--info}` class.
{: .notice--info}

**Watch out!** This paragraph of text has been [emphasized](#) with the `{: .notice--warning}` class.
{: .notice--warning}

**Watch out!** This paragraph of text has been [emphasized](#) with the `{: .notice--success}` class.
{: .notice--success}

**Watch out!** This paragraph of text has been [emphasized](#) with the `{: .notice--danger}` class.
{: .notice--danger}

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/chakrabar/chakrabar.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
