---
title: "jerryPycia"
date: 2020-05-25T11:12:01+01:00
draft: false
description: "A Grateful Dead python library to explore show data" 
tags: ["python", "pip", "Grateful Dead"]
categories: ["Python", "GratefulDead", "Music"]
---

## What a long strange pip its been...

Grateful Dead have performed around 2500 shows. Remarkably, around 2200 of these have been recorded. To explore (most of) this data I have created a python package called jerryPycia. I think thats a really funny name. I'm also pretty happy with my "long strange pip" joke. You can install it using pip, like:

```
pip install jerryPycia
```

Its also available on my [Github](https://github.com/andrewblance/jerryPycia) page.

Once you have it installed, you can use it to do a few things. But first:

```
import jerrypycia as jPy
data, raw = jPy.grateful_loader()
```
We now have two objects: ```data``` and ```raw```. The latter is just the raw data, in the form of a Pandas DataFrame. I'll explain later, but it's not every show they've played, but its a lot of them.

The first object, ```data``` is a object I've created. I made a class ( ```Dataset```, which is found in ```jerryPycia.py``` in the git repo) which "holds" the csv. What else this class does is give us access to a few methods. These methods allow us to very quickly investigate the data. I'm not sure I'm totally happy with this structure. However, it means that you don't need to pass anything around, and hopefully it should just work! It also solved another problem I was facing, which I will discuss soon.

## Methods

### .randomShow()

The first method will query the data and return to us a random show they played. 

{{< cupperFig
img="rS.JPG"
caption="random gig"
command="Resize"
options="2000x" >}}

### .nextShow()

The second method will iterate through the data. Running ```print(data.nextShow())``` will give you the first show they played (that's available in my dataset), then if you run it again it will give you the second, and so on. This is done using ```try```, ```next``` and ```except```. I think it's a pretty neat syntax.

```{python}
def nextShow(self):
    """
    iterate through the gigs
    return as the declared Show() class
    """
    it = iter(self.iter)
    try:
        elem = next(it)
        return Show(elem[1])
    except StopIteration:
        print("End of list of gigs!")
        return
```

To do this you cannot just pass it a DataFrame, it has to be modified using ```.iterrows()``` (which is what ```self.iter``` is). I didn't want the user to have to deal with creating and managing the two different datasets (the "iterable" version and the non-iterable) one. Therefore, I thought I would merge it into one class, and let that handle it all.

### .search_show()

The final method returns a few things. If you pass it a song title, it will give you the first and last time they performed the song, how many times it was performed and a plot including everytime they played it. If you run this command: ```data.song_search("Drums", plot=True)``` you get the following output:

{{< cupperFig
img="sS"
caption="song search!"
command="Resize"
options="2000x" >}}

Potentially, in the future, I may add something that could output maps or write everything to a pdf report.

## Data

I was unable to find a single file that included all the data I would need (though there could be one out there!). To solve this I scraped the data of off this [website](https://www.cs.cmu.edu/~mleone/gdead/setlists.html). Getting the data scraped was an easy process compared to cleaning it (though it was also not that easy). Though the data looks very similar when you go through the site there is actually a lot of small differences. Different years are treated differently a lot of the time. Sometimes it includes who opened the show, or if the did an encore, or it won't include the city (or state or venue or country) they performed in and lots of other things. This all added up to it being non-trivial to fit it all into a nice, clean csv file as all cases needed to be considered differently. 

On top of this, this source only includes show from 1972, which means my dataset misses out around 600 shows as far as I can tell. Hopefully in the future I can add these other shows in and add more information to each of them.

## Poetry

Poetry is a tool to build packages in a nice, easy way. This is what I used take my code and put it on pip. Poetry will create a structure like this:

```
/jerryPycia
├── README.md
├── jerrypycia
│   └── __init__.py
│   └── jerryPycia.py
│   └── datasets
│       └── GratefulDead.csv
├── pyproject.toml
└── tests
    ├── __init__.py
    └── test_jerrypycia.py
```

```pyproject.toml``` contains what the dependancies are and other general information about your package. All of the "code" is in ```jerryPycia.py```

Poetry also adds a easy way to perform tests on your code. At the moment I have not added any to this project, but that would be an obvious next step to take.

## Reddit 

I asked in Reddit and stack overflow how to improve my code, and got a few responses. One of them told me about the existence of this [repo](https://github.com/jefmsmit/gdshowsdb). I never saw this when I was trying to collect my data, but in it is a database of every performance! Very useful! I may try to incorporate this database into my library as well.

## Conclusion

download my library.
