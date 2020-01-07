---
layout: post
title:  "revamp selenium: refine the way you use parent-child selectors"
date:   2020-01-06 10:20:00 +0200
categories: "revamp selenium"
---
**always keep in mind that automated tests should work for you...** and should provide enough details on execution failures so all the involved parties spend less time analyzing failures, debugging and fixing problems!

## the statement 
imagine that we have the following DOM struct.
```html
<ul class="post-list">
  <li>
    <a class="post-link" href="/testing/2019/12/18/broken-links-with-python.html">
      testing with python scrapy</a>
  </li>
  <li>
    <a class="post-link" href="/testing,/mocha,/javascript/2019/12/09/webdriverio-and-ddt.html">
      data driven testing in Mocha Js</a>
  </li>
</ul>
```
and lets say that for locating the first `<a>` element on the list you decide to use a simple css locator: `.post-list .post-link` or xpath: `//ul[@class='post-list']//a[@class='post-link']`.

when you run the following python script:

```python
from selenium import webdriver
driver_location = './chromedriver'

driver = webdriver.Chrome(driver_location)
driver.get('https://pjcalvo.github.io')

parent_child_locator = '.post-list .post-link' #combined parent child locator
element = driver.find_element_by_css_selector(parent_child_locator)

print(f'Element was found with href: { element.get_attribute("href")}')
driver.close()
```

you get a console message like this:<br>
`Element was found with href: https://pjcalvo.github.io/testing/2019/12/18/broken-links-with-python.html`

## the problem
in case that one of the classes changed, and you re-run the script: <br>
`Message: no such element: Unable to locate element: {"method":"css selector","selector":".post-lists .post-links"}`

something happened!! but the message above is not really granular, we are using the class value of **two different elements** and so **we can't** figure out which class changed just by looking at the message. we will need to investigate, debug and tweak. 

## solution
**let's change out approach on how we locate this element** 
first we are going to search the parent element, then execute a second search on the child element. Therefore we can easily and quickly understand what changes happened to our system.

```python
from selenium import webdriver
driver_location = './chromedriver'

driver = webdriver.Chrome(driver_location)
driver.get('https://pjcalvo.github.io')

parent_locator = '.post-list' #only the parent
child_locator = '.post-links' #only the child

#search the parent then search the child
element = driver.find_element_by_css_selector(parent_locator) \
                .find_element_by_css_selector(child_locator) 

print(f'Element was found with href: { element.get_attribute("href")}')
driver.close()
```

**in case of success the outcome is the same:**<br>
`Element was found with href: https://pjcalvo.github.io/testing/2019/12/18/broken-links-with-python.html`

**but in case of failure:**<br>
`Message: no such element: Unable to locate element: {"method":"css selector","selector":".post-links"}`

very fast we can identify that the problem is searching the child element, *and the parent search is still working fine.*


## final thoughts
this is a quite and easy sample, but i feel that the point is also quite easy to understand. always be mindful about how you use css and xpath locators, you might be using them quite well, but as with any other system it is recommended to stop for a second, look at the code and re-factor in case of misuses.

there is something else to consider and give a thought. When you add more searches to the test you will also affect the performance of your execution; each search in this case is a new call to the selenium server. I am a fan of having tests working for me even when I have to write an extra line of code.

Comments?

cheers :) and remember to keep revamping your tests
[https://pjcalvo.github.com](https://pjcalvo.github.com)