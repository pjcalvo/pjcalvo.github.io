---
layout: post
title:  "revamp selenium: The power of explicit waits"
date:   2020-01-17 10:20:00 +0200
categories: "revamp selenium"
---
Working and testing with selenium will make us very patient developers. We fill our code with a lot of waits all the time, and is natural and is expected.

UI tests have a huge dependency on the external conditions of the system under tests, for example: rendering speed, internet connection and third party tools. Also sometimes we need to wait because it is how our system behaves and **we need certain conditions to be full-filled** before our test can continue executing. 

Selenium provides 2 ways of waiting (+1 non selenium):

* Implicit wait: Just gives the driver a range of time to **find an element** before breaking the execution. In natural words: *Hey selenium if you can't find my elements in 30 seconds please finish the test*.
* Explicit wait: Verifies for conditions of an element to be full-filled before breaking or continue with the execution. *Hey selenium, find my element but also make sure it has <thisclass> as a class before continue*.
* Sleep: We sleep our execution thread, then proceed. *Hey selenium STOP . . . . . . . . please proceed.*

 
## Talking about explicit waits
Imagine we are working with a [countdown website](https://gcalvocr.github.io/countdown-app/), and we want to make sure that the counter actually works.

```python
driver = webdriver.Chrome(DRIVER_LOCATION) #driver
driver.get("https://gcalvocr.github.io/countdown-app/")

print('**** Click on 1 Minute ****')
driver.find_element_by_id("btn-1").click() # click on start 1 minute countdown
```

At this point we could be tempted to add a 60 seconds hardcoded wait and they verify that the counter is 00:00.

```python
time.sleep(60000)
if driver.find_element_by_id('countdown').text == '00:00':
  print('Yes, the counter worked')
else:
  print('Nop, The expected value is not correct')
```

And the solution works as a charm.
![Worked](https://thepracticaldev.s3.amazonaws.com/i/12dc5jx437ourpl781o6.png).

**But**, what if it fails? We would have to wait 1 entire minute to figure it out. What if our element is not visible anymore after 2 seconds? What if the counter never started?

## Let's create a custom explicit wait

```python
class element_has_text(object):
  def __init__(self, locator, text): #python contructor
    self.locator = locator
    self.text = text

  def __call__(self, driver):
    element = driver.find_element(*self.locator) # find the element
    if element.text == self.text: # if the element text is expected return true
        return True
    else:
        return False
```

The code above extends the WebDriverWait class to add a new custom wait. This custom wait is going to verify constantly for the condition `if element.text == self.text`, but will break the test only when the given time is burned down and the condition was not met. 

So the new code could look like this using the **WebDriverWait** and the custom **element_has_text**.

 ```python
  print('**** Click on 1 Minute ****')
  driver.find_element_by_id("btn-1").click() # click on start 1 minute countdown

wait = WebDriverWait(driver, 10) # wait max 10 seconds
locator = (By.ID, 'countdown') # element locator
wait.until(element_has_text(locator, '00:55'))
print('**** Timer has 55 seconds left ****')
wait.until(element_has_text(locator, '00:45'))
print('**** Timer has 45 seconds left ****')
wait.until(element_has_text(locator, '00:35'))
print('**** Timer has 35 seconds left ****')
# this will break now
wait.until(element_has_text(locator, '00:10'))
```

And the outcome is a sweet *selenium.common.exceptions.TimeoutException* that we can handle as part of the test failure:

![Sweet expection](https://thepracticaldev.s3.amazonaws.com/i/ks661f3xsn2txxcq46mo.png)

This is quite more powerful, because it is also looking for the element and verifying a property. In case the element can't be found the test will also break.

## Final thoughts

Selenium has several built in explicit waits that will adapt to most of the use cases, so there might no be need for you to create a custom one. But I wanted to go a step further and demonstrate that you can easily extend your testing framework and adapt to your system under test.

Note: I don't want to say that `sleep.wait()` is the devil and that you should try to avoid it, but yes it is the devil and you should avoid it as much as possible.

Please take your time to read the selenium documentation and reach out if you need any advice on how to improve your code quality and tests speed.

[Waits](https://selenium-python.readthedocs.io/waits.html#implicit-waits)

Comments?

cheers :) and remember to keep waiting
