Python-Goose - Article Extractor |Build Status|
===============================================

Warning
-----

This is a modified version of Goose meant to be used internally with Google App Engine (GAE) application.
The image processing features for determining image file type, size and additional attributes have been
removed from this version if running in an environment with no file system. Drop me a note if you need these
features implemented and i might be able to provide some insights as i have been trying to make these parts
functional but was wasting too much time on it without actually needing it. Sorry for that :/

Intro
-----

Goose was originally an article extractor written in Java that has most
recently (aug2011) converted to a scala project by Gravity.com

This is a complete rewrite in python. The aim of the software is is to
take any news article or article type web page and not only extract what
is the main body of the article but also all meta data and most probable
image candidate.

Goose will try to extract the following information:

-  Main text of an article
-  Main image of article
-  Any Youtube/Vimeo movies embedded in article
-  Meta Description
-  Meta tags

Originally, Goose was open sourced by Gravity.com in 2011

-  Lead Programmer: Jim Plush (Gravity.com)
-  Contributers: Robbie Coleman (Gravity.com)

The python version was rewrite by:

-  Xavier Grangier (Recrutae.com)

Licensing
---------

If you find Goose useful or have issues please drop me a line, I'd love
to hear how you're using it or what features should be improved

Goose is licensed by Gravity.com under the Apache 2.0 license, see the
LICENSE file for more details

Setup
-----

::

    mkvirtualenv --no-site-packages goose
    git clone https://github.com/grangier/python-goose.git
    cd python-goose
    pip install -r requirements.txt
    python setup.py install

Take it for a spin
------------------

::

    >>> from goose import Goose
    >>> url = 'http://edition.cnn.com/2012/02/22/world/europe/uk-occupy-london/index.html?hpt=ieu_c2'
    >>> g = Goose()
    >>> article = g.extract(url=url)
    >>> article.title
    u'Occupy London loses eviction fight'
    >>> article.meta_description
    "Occupy London protesters who have been camped outside the landmark St. Paul's Cathedral for the past four months lost their court bid to avoid eviction Wednesday in a decision made by London's Court of Appeal."
    >>> article.cleaned_text[:150]
    (CNN) -- Occupy London protesters who have been camped outside the landmark St. Paul's Cathedral for the past four months lost their court bid to avoi
    >>> article.top_image.src
    http://i2.cdn.turner.com/cnn/dam/assets/111017024308-occupy-london-st-paul-s-cathedral-story-top.jpg

Configuration
-------------

There is two way to pass configuration to goose. The first one is to
pass to goose a Configuration() object. The second one is to pass a
configuration dict

For instance, if you want to change the userAgent used by Goose juste
pass :

::

    >>> g = Goose({'browser_user_agent': 'Mozilla'})

Switching parser : Goose can now be use with lxml html parser or lxml
soup parser. By default the html parser is used. If you want to use the
soup parser passe it in the configuration dict :

::

    >>> g = Goose({'browser_user_agent': 'Mozilla', 'parser_class':'soup'})

Goose is now language aware
---------------------------

For exemple scrapping a spanish content page with correct meta language
tags

::

    >>> from goose import Goose
    >>> url = 'http://sociedad.elpais.com/sociedad/2012/10/27/actualidad/1351332873_157836.html'
    >>> g = Goose()
    >>> article = g.extract(url=url)
    >>> article.title
    u'Las listas de espera se agravan'
    >>> article.cleaned_text[:150]
    u'Los recortes pasan factura a los pacientes. De diciembre de 2010 a junio de 2012 las listas de espera para operarse aumentaron un 125%. Hay m\xe1s ciudad'

Some pages don't have correct meta language tags, you can force it using
configuration :

::

    >>> from goose import Goose
    >>> url = 'http://www.elmundo.es/elmundo/2012/10/28/espana/1351388909.html'
    >>> g = Goose({'use_meta_language': False, 'target_language':'es'})
    >>> article = g.extract(url=url)
    >>> article.cleaned_text[:150]
    u'Importante golpe a la banda terrorista ETA en Francia. La Guardia Civil ha detenido en un hotel de Macon, a 70 kil\xf3metros de Lyon, a Izaskun Lesaka y '

Passing {'use\_meta\_language': False, 'target\_language':'es'} will
force as configuration will force the spanish language


Video extraction
----------------

::

    >>> import goose
    >>> url = 'http://www.liberation.fr/politiques/2013/08/12/journee-de-jeux-pour-ayrault-dans-les-jardins-de-matignon_924350'
    >>> g = goose.Goose({'target_language':'fr'})
    >>> article = g.extract(url=url)
    >>> article.movies
    [<goose.videos.videos.Video object at 0x25f60d0>]
    >>> article.movies[0].src
    'http://sa.kewego.com/embed/vp/?language_code=fr&playerKey=1764a824c13c&configKey=dcc707ec373f&suffix=&sig=9bc77afb496s&autostart=false'
    >>> article.movies[0].embed_code
    '<iframe src="http://sa.kewego.com/embed/vp/?language_code=fr&amp;playerKey=1764a824c13c&amp;configKey=dcc707ec373f&amp;suffix=&amp;sig=9bc77afb496s&amp;autostart=false" frameborder="0" scrolling="no" width="476" height="357"/>'
    >>> article.movies[0].embed_type
    'iframe'
    >>> article.movies[0].width
    '476'
    >>> article.movies[0].height
    '357'


Goose in Chinese
----------------

Some users want to use Goose for chinese content. Chinese word
segementation is way more difficult to deal with that occidental
languages. Chinese needs a dedicated StopWord analyser that need to be
passed to the config object

::

    >>> from goose import Goose
    >>> from goose.text import StopWordsChinese
    >>> url  = 'http://www.bbc.co.uk/zhongwen/simp/chinese_news/2012/12/121210_hongkong_politics.shtml'
    >>> g = Goose({'stopwords_class': StopWordsChinese})
    >>> article = g.extract(url=url)
    >>> print article.cleaned_text[:150]
    香港行政长官梁振英在各方压力下就其大宅的违章建筑（僭建）问题到立法会接受质询，并向香港民众道歉。

    梁振英在星期二（12月10日）的答问大会开始之际在其演说中道歉，但强调他在违章建筑问题上没有隐瞒的意图和动机。

    一些亲北京阵营议员欢迎梁振英道歉，且认为应能获得香港民众接受，但这些议员也质问梁振英有

Goose in Arabic
---------------

In order to use Goose in Arabic you have to use the StopWordsArabic
class.

::

    >>> from goose import Goose
    >>> from goose.text import StopWordsArabic
    >>> url = 'http://arabic.cnn.com/2013/middle_east/8/3/syria.clashes/index.html'
    >>> g = Goose({'stopwords_class': StopWordsArabic})
    >>> article = g.extract(url=url)
    >>> print article.cleaned_text[:150]
    دمشق، سوريا (CNN) -- أكدت جهات سورية معارضة أن فصائل مسلحة معارضة لنظام الرئيس بشار الأسد وعلى صلة بـ"الجيش الحر" تمكنت من السيطرة على مستودعات للأسل

TODO
----

-  Video html5 tag extraction

Known issues
------------

-  There is some issue with unicode URLs.

OS X 10.7 Install Instructions
------------------------------

Installation Help:

1. Install python-devel if you don't have it
2. Install libjpeg brew install libjpeg

3. You need to install the python imaging library. We wont be using it,
   but its a dependency deep in the goose egg (fun!).

a. download

   ::

       curl -O -L http://effbot.org/downloads/Imaging-1.1.7.tar.gz

b. extract

   ::

       tar -xzf Imaging-1.1.7.tar.gz
       cd Imaging-1.1.7

c. build and install

   ::

       python setup.py build
       sudo python setup.py install

4. Next up clone this repo and install the egg.

5. Once you install the egg you have to then copy the resources
   directory manually into the egg. There is something screwy about the
   way its setup.

.. |Build Status| image:: https://www.travis-ci.org/xgdlm/python-goose.png?branch=master
   :target: https://www.travis-ci.org/xgdlm/python-goose
