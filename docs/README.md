# Welcome to Plutus Community Docs 

### Disclaimer
---
<blockquote>
  <span style="color:lightorange;font-weight:700;font-size:16px">
    <p>This site is for the benefit of the Cardano community and is owned and maintained by the community under an open source philosophy. Input Output Global, Inc. (IOG) is not responsible for, and makes no representations or warranties regarding, the accuracy, reliability and completeness of the aggregated content found in this site. Any use of the open source Apache 2.0 licensed software is done at your own risk and on a “AS IS” basis, without warranties or conditions of any kind.</p>
  </span>
</blockquote>
---


This documentation site is built from Cardano community contributions. It started as a knowledge-sharing site for Plutus Pioneer students and alumni. As the community grows the scope of the content here also follows a proportional path. 

Our mission is to help future Cardano developers stand on the shoulder of giants by provding a rich resource that perfects itself after each iteration of educational programs and each wave of dApps that raises the ecosystem's standards.

**Important:** Please don't expect that everything will work out of the box as the Plutus language is in its infancy and evolves rapidly. All the content here is created from the best efforts and knowledge of volunteers. 

Having said that, we always aim to improve so contributions are welcomed. Before you raise a Pull Request see the [How to Contribute](https://github.com/input-output-hk/plutus-community/blob/main/CONTRIBUTING.md){:target="_blank"} guide. 



This website has been a valuable source of information for past cohorts, mainly because:

-   Plutus development has evolved since the first cohort and will likely keep evolving
-   official documentation is often out of date or fragmented
-   new comers are introducted to new tools and techniques
-   the environment setup instructions may vastly differ between platforms
-   setting up an environment for Plutus development can be challenging

### How to collaborate

Any collaboration is welcome, it doesn't need to be a new document, corrections to the old ones are much needed. To do so, just modify or create a document in the corresponding directory inside "docs", and make a pull request using the community repository:

<https://github.com/plutus-community/docs/>

In the case of contributing a fix, please make sure to include a description of the issue fixed to facilitate the task of the moderator. Once it is approved, it'll be automatically published and linked into the ToC of the site (for now, using readthedocs.io service thru mkdocs).

### Style Guide

At the momment we are using Markdown language for the documentation for its simplicity, broad adoption and syntax support within the Github web-based editor. Additionally, we're using Github for version and access control so the combination works seamlessly well together. More information on Markdown can be found here:

<https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax>

To help standarize the content, we recommend to write the articles following the schema below:

-   Title: using #, must be short and descriptive, including OS and technology when applicable. Beware that this string will be used by mkdocs to generate the navigation menu, so try to be as specific as possible.
-   Author, contact, date of creation, date of last revision.
-   Brief description.
-   Sections (##) Subsections (###) and content: as a general rule, avoid using pictures unless it is strictly necessary. If you do, upload it to the "img" directory, and name it using snake_case_format_XX.ext using the title of the article for easy identification. Use much plain text as possible, it is the preferred option as it can be copied, pasted, and searched internally and externally to the site. Make use of \``` \``` for marking code snippets.
-   Changelog (optional): Brief summary of changes over time.
-   Link to a dedicated forum thread (optional) : Link to a new post on Cardano's forum announcing the new article, that can be used to interact with other users, overcoming this way the static nature of the documentation site.

### Other resources

Plutus Pioneers Course Materials:  
<https://github.com/input-output-hk/plutus-pioneer-program>

Learn You a Haskell introduction:  
<http://learnyouahaskell.com/>

Haskell and Cryptocurrencies course:  
<https://www.youtube.com/playlist?list=PLJ3w5xyG4JWmBVIigNBytJhvSSfZZzfTm>

Discord:

Cardano Forum:  
<https://forum.cardano.org/>

Stack Exchange:  
<https://cardano.stackexchange.com/>

Cardano Documentation:  
<https://docs.cardano.org/>

Developer Portal:  
<https://developers.cardano.org/>

IOHK Youtube  
<https://www.youtube.com/channel/UCBJ0p9aCW-W82TwNM-z3V2w>

Charles Hoskinson's Youtube  
<https://www.youtube.com/channel/UCiJiqEvUZxT6isIaXK7RXTg>

On Github

-   IO Top-Level <https://github.com/input-output-hk>

-   Cardano-node <https://github.com/input-output-hk/cardano-node>

-   Plutus <https://github.com/input-output-hk/plutus>

-   Plutus Applications <https://github.com/input-output-hk/plutus-apps>

-   Plutus Docs <https://plutus-apps.readthedocs.io/>

-   Marlowe <https://github.com/input-output-hk/marlowe-cardano>
