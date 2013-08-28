##webalchemy: realtime, Pythonic web framework
webalchemy is a fast, simple and lightweight realtime micro web-framework for Python, inspired by [SQLAlchemy](http://www.sqlalchemy.org/), the [IPython notebook](http://ipython.org/), and of course [Meteor](http://www.meteor.com/). With webalchemy you develop webapps like you would develop a desktop (or mobile) app

####Example
We "translated" Meteor colors app to webalchemy. The app below can be seen in action [here](https://vimeo.com/73073766) and the Meteor original [here](http://www.meteor.com/screencast)
```python
from tornado import gen
from webalchemy import server
from webalchemy.widgets.basic.menu import menu

class colors_app:    

    # shared state between sessions
    colors_count={
        'foo'             :0,
        'bar'             :0,
        'wowowowowo!!!'   :0,
        'this is cool'    :0,
        'WEBALCHEMY ROCKS':0,
    }

    # this method is called when a new session starts
    @gen.coroutine
    def initialize(self, remotedocument, wshandler, message):
        # remember these for later use
        self.rdoc= remotedocument
        self.wsh= wshandler

        self.increase_count_by= self.rdoc.jsfunction('element','amount','''
            element.app.clickedcount+= amount;
            if (element.app.clickedcount>0.5) {
                element.textContent= '('+element.app.clickedcount+') '+element.app.text;
            }''')

        # do this for each element added to the menu
        def on_add(element,color):
            element.att.app.text= color
            element.att.app.clickedcount= colors_app.colors_count[color]
            # note below: inline interpolation of increase_count_by,
            # and rpc call from js to Python. It is called 'srpc'
            # since it goes for all sessions
            element.att.onclick= self.rdoc.jsfunction('event','''
                srpc('serverside_on_button_clicked', event.target.app.text);
                #{self.increase_count_by}(event.target,1);
            ''')
            # calling this just to update the count in the item text
            self.increase_count_by(element,0)

        # the menu, with some styling
        self.menu= menu(self.rdoc, on_add)
        self.menu.rule_menu.att.style(display='table',margin='10px')
        self.menu.rule_item.att.style(
            color='#000000',
            fontSize='2em',
            textTransform='uppercase',
            fontFamily='Arial, Verdana, Sans-serif',
            float='bottom',
            padding='10px',
            listStyle='none',
            cursor='pointer',
            webkitTransition='all 0.3s linear',
            webkitUserSelect='none'
        )
        self.menu.rule_item_hover.att.style(
            color='#ffffff',
            background='#000000',
            paddingLeft='20px',
            webkitTransform='rotate(5deg)'
        )
        # populate the menu...
        self.rdoc.body.append(self.menu.element)
        for color in colors_app.colors_count:
            self.menu.add_item(color)

    # register the method so we can call it from frontend js directly
    # this could also be done by messaging, see in simple_examplee.py
    @server.rpc
    @gen.coroutine
    def serverside_on_button_clicked(self, sender_doc, color):
        colors_app.colors_count[color]+= 1
        if sender_doc is not self:
            for e in self.menu.element.childs:
                if e.text==color:
                    self.increase_count_by(e,1)

if __name__=='__main__':
    server.run('localhost',8083,colors_app) # the first parameter is the port...
```
##Philosophy and Roadmap
Run relatively simple code in the frontend, and structure this code so it is easy to automate from the backend. I would like to have a rich widgets library and a strategy for easy development of new ones. The roadmap to achieve this is to use webalchemy to reimplement demos and examples from Meteor and other frameworks, while filling in missing functionality.

##Requirements
* Python >= 3.3
* Tornado >= 3.1
* webalchemy webapps require a modern webbrowser supporting websockets (too much web in this sentence)

##License: MIT
