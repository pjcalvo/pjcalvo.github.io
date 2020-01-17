---
layout: post
title:  "Playing around with selenium and openCV!"
date:   2020-01-15 10:20:00 +0200
categories: "revamp selenium"
---
**Selenium library is frequently used for UI test automation, and was probably designed for it, but selenium itself is not a testing library and people quite often forget about it**

Selenium is basically a set of tools that allow your code to interact with a browser and the html elements on it.

The selenium WebDriver architecture consists in four main parts: 
- The language bindings (libraries for each language).
- The selenium server.
- The browser drivers.
- Finally, the drivers.

## Playing around with selenium

Selenium is not a super fast processing tool, maybe because of the many components that need to get involved. I made a fun experiment to see how slow is really selenium by reducing the external dependencies at its minimum. 

For the experiment I will basically be executing clicks against a system in which no internet call is executed on each click and also there is no html rendering happening (or this does not matter that much).

### Python and open CV
I used python along with openCV to open an image and generate a list of pixel points for each black pixel on the given image.

```python
def generate_coordenates():
    print('Generating coordinates based on image')
    image=cv2.imread('image2.png')
    black_coordinates = [] #initial position

    for i in range(image.shape[0]):
        for j in range(image.shape[1]):
            # find the black pixels
            if image[i,j,0]==0 and image[i,j,1]==0 and image[i,j,2]==0: 
                black_coordinates.append((j,i))

    print('Writing coordinates to file')
    # generate a coordinates file
    with open(COORDINATES_FILE, 'a') as out:
        for x, y in black_coordinates:
            out.write(f'{x},{y}\n')
    
    print('Finish generating coordinates file')

```

### Selenium and a website like `MS Paint`
Then I use selenium to go to a website like MSPaint (https://kleki.com/) and generate an image based on the black pixels.

```python
driver = webdriver.Chrome(DRIVER_LOCATION)
        with open(COORDINATES_FILE) as file:
            driver.get(DRAW_WEBSITE)
            driver.find_element_by_css_selector("html")
            
            # move to starting point
            actions = ActionChains(driver)
            actions.move_by_offset(STARTING_COORDINATES.get('x'), STARTING_COORDINATES.get('y'))
            # start painting
            for line in map(lambda line: line.rstrip('\n'), file):
                print('Creating move actions')
                xy = line.split(',') # split the x and y
                current = {'x': int(xy[0]), 'y': int(xy[1])} 
                move_to = {'x': current.get('x') - before.get('x'), 
                           'y': current.get('y') - before.get('y')}

                actions.move_by_offset(move_to.get('x'), move_to.get('y'))
                actions.click()
                before = current.copy()

            # finish painting
            actions.perform()
```

### The process

A couple of minutes:
![a couple of minutes](https://thepracticaldev.s3.amazonaws.com/i/y7r0qxd9fbs7glllffto.png)

20 minutes:
![15 minutes](https://thepracticaldev.s3.amazonaws.com/i/sy7z9565pj9kl0t0t1wl.png)

50 minutes:
![50 minutes](https://thepracticaldev.s3.amazonaws.com/i/yk96h8mx0s3kjhqbcd4w.png)


### The results
The coordinates file ended up with **3777 rows**. Because for each row selenium executes 2 actions: `move_by_offset` and `click` it means **7554 selenium actions** were be executed.

The test took **54.67 minute** to complete which means around **3280 seconds**.

And if we do the math: *7554 / 3280 = ~2.30*. It means that 2.30 selenium actions were executed per second.

Quite fun... 


Comments?

cheers :) and remember to keep using selenium
