# Sprint 1: Reading Web-News Sources
### (Sprint Concept: Working with Data Formats & Terminology)
11/16/17

**Author:** **Jon Honda**


__*Project Description:*__ This project sprint is an initial exploration of a larger project that will identify how different web-news sources report on the same story. The initial exploration includes:
1.  Introduce myself to python, Atom text editor, project sprint concept, and development documentation procedures.
2.  Identify HTML standards and commonalities used by 2 or 3 web sources.
3.  Identify and explore python scraping libraries.

__*Learning Summary:*__ Primary project objectives were achieved. Additional items of value were also learned.
Primary objectives:

**Objective 1:** Introduce myself to python:

    Successfully installed python and ran a Hello World Program.
    Successfully installed the Newspaper python library
    Used Jupyter notebook to document items learned, and process I went through.
    Experienced a 2-week development sprint that started with discussion on data formats and terminology; went through project selection and refinement process; project implementation; and presentation and wrap up.

**Objective 2:** Identify HTML standards and commonalities used by web sources.

    I examined news articles from 2 sources: APNews and FoxNews.

    I identified a seemingly common HTML standard for encoding web-news summaries. The encodement is a JSON data type located inside as script tag in the HTML head tag:  <script type ="application/ld+json">
    The encodement contains items such as article title, author, and date.

    Article body text does not appear to have a well standardized format. However, I did notice that article text was always located in paragraph tags inside the HTML body tag.

**Objective 3:** Identify and explore python scraping libraries

    I explored the Newspaper python library.
    The library will parse a news article into compenents such as article title, author, date, url, and article text.
