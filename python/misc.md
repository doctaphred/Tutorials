hasattr in Python < 3.2 swallows exceptions -.-

http://stackoverflow.com/questions/903130/hasattr-vs-try-except-block-to-deal-with-non-existent-attributes/16186050#16186050
http://bugs.python.org/issue2196
http://hg.python.org/cpython/rev/4c8375574aa9/
http://bugs.python.org/issue9666


    class Test(object):
        @property
        def prop(self):
            if random.random() < 0.5:
                 raise ValueError
             else:
                 return 'bogus'
    
    
    In [25]: hasattr(Test(), 'prop')
    Out[25]: False
    
    In [26]: hasattr(Test(), 'prop')
    Out[26]: False
    
    In [27]: hasattr(Test(), 'prop')
    Out[27]: False
    
    In [28]: hasattr(Test(), 'prop')
    Out[28]: False
    
    In [29]: hasattr(Test(), 'prop')
    Out[29]: True
    
    In [30]: hasattr(Test(), 'prop')
    Out[30]: True
