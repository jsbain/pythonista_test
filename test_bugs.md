# Pythonista bugtests
This file documents a few pythonista bugs, mostly to simplify testing between versions.  run using 
```
import doctest
doctest.testfile('test_bugs.md')
```

# version (not a bug)
This is a stripped down version of pythonista_version.py from ccc.  Originally i was downloading this on the fly... but i couldn't get the code working in p3

```python
>>> import editor
>>> editor_oldfile=editor.get_path()
>>> import os, platform, plistlib, scene, sys
>>> def pythonista_version():  
...    plist = plistlib.readPlist(os.path.abspath(os.path.join(sys.executable, '..', 'Info.plist')))
...    return '{CFBundleShortVersionString} ({CFBundleVersion})'.format(**plist)
>>> ios_ver, _,  machine_model = platform.mac_ver()
>>> bit = platform.architecture()[0].rstrip('bit') + '-bit'
>>> rez = '({:.0f} x {:.0f})'.format(*scene.get_screen_size())
>>> fmt = 'Pythonista version {} on iOS {} on a {} {} with a screen size of {} * {:.0f}'
>>> print(fmt.format(pythonista_version(), ios_ver, bit, machine_model, rez, scene.get_screen_scale())) 
Pythonista version info here (this is not an error)

``` 

# convert point: fullscreen
convert point and convert rect give values scaled incorrectly and relative to wrong corner, perhaps orientation dependent,when one view is None.  The current behavior seems... broken in new and different ways from the old behavior

```python
>>> import ui,time
>>> v=ui.View()
>>> v.present()
>>> time.sleep(1)
>>> ui.convert_point((10,0), v,None)-ui.convert_point((0,0), v,None)
Point(10.00, 0.00)
>>> ui.convert_point((0,10),v,None)-ui.convert_point((0,0), v,None)
Point(0.00, 10.00)
>>> v.close()
>>> # test roundtrip conversion tofrom None 
>>> ui.convert_point(ui.convert_point((11,12),v,None),None,v)
Point(11.00, 12.00)

```
# convert point:sheet
```python 
>>> time.sleep(1)
>>> import ui,time
>>> v=ui.View()
>>> v.present('sheet')
>>> time.sleep(1)
>>> ui.convert_point((10,0), v,None)-ui.convert_point((0,0), v,None)
Point(10.00, 0.00)
>>> ui.convert_point((0,10),v,None)-ui.convert_point((0,0), v,None)
Point(0.00, 10.00)
>>> ui.convert_point(ui.convert_point((11,12),v,None),None,v)
Point(11.00, 12.00)
>>> v.close()
>>> time.sleep(1)
>>> v.on_screen
False

``` 

# convert point:panel
```python
>>> time.sleep(1)
>>> import ui,time
>>> v=ui.View()
>>> v.present('panel')
>>> time.sleep(1)
>>> ui.convert_point((10,0), v,None)-ui.convert_point((0,0), v,None)
Point(10.00, 0.00)
>>> ui.convert_point((0,10),v,None)-ui.convert_point((0,0), v,None)
Point(0.00, 10.00)
>>> ui.convert_point(ui.convert_point((11,12),v,None),None,v)
Point(11.00, 12.00)

``` 
## panel view close
close should close the tab if presented as a tab! 

```python
>>> v.close()
>>> time.sleep(1)
>>> v.on_screen
False

``` 
# popover
```python
>>> time.sleep(1)
>>> import ui,time
>>> v=ui.View()
>>> v.present('popover')
>>> time.sleep(1)
>>> ui.convert_point((10,0), v,None)-ui.convert_point((0,0), v,None)
Point(10.00, 0.00)
>>> ui.convert_point((0,10),v,None)-ui.convert_point((0,0), v,None)
Point(0.00, 10.00)
>>> ui.convert_point(ui.convert_point((11,12),v,None),None,v)
Point(11.00, 12.00)
>>> v.close()
>>> time.sleep(1)
>>> v.on_screen
False

``` 
## popover_location does not accept ui.Point
most other View methods were updated to allow this, but not present.  use case is converting from a touch.location to a popover, etc.  

```python
>>> v.present('popover',popover_location=ui.Point(0,0))
>>> v.close()

``` 
# touch enabled
simply cannot be changed 

```python
>>> class CustomView(ui.View):
...	def __init__(self):
...		pass
>>> v=CustomView()
>>> v.touch_enabled
True
>>> v.touch_enabled=False
>>> v.touch_enabled
False

``` 
# editor newline mangling
editor strips trailing newline from files

```python
>>> import os, time
>>> import tempfile
>>> tmpfile=tempfile.NamedTemporaryFile(delete=False)
>>> with tmpfile as f:
...	f.write('a\n')
>>> import editor
>>> editor.open_file(tmpfile.name)
>>> time.sleep(1)
>>> editor.replace_text(0,1,'b')
>>> time.sleep(1)
>>> with tempfile.NamedTemporaryFile(delete=True) as f:
...   editor.open_file(f.name)
...   time.sleep(1)
>>> time.sleep(1)
>>> open(tmpfile.name).read()
'b\n'
>>> editor.open_file(editor_oldfile)
>>> os.remove(tmpfile.name)

``` 
# View children constructors
## tableview constructor arguments
Tableview does not seem to support flex or frame in its constructor

```python
>>> v=ui.TableView(frame=(0,0,200,200), flex='h')
>>> v.flex
'H'
>>> v.frame
Rect(0.00, 0.00, 200.00, 200.00)

``` 

# scene things
## ShapeNode docs implies path can be None (otherwise, path should bewuired, not optional)
```python
>>> import scene
>>> scene.ShapeNode() #doctest:+ELLIPSIS
<...ShapeNode...

``` 

## SpriteNode texture=None
SpriteNode.texture docs says about texture:

> If the value is None, the sprite is drawn as a colored rectangle using its color attribute. Otherwise, the texture is used to draw the sprite.

```python
>>> scene.SpriteNode(texture=None,color=(0,1,1)) #doctest:+ELLIPSIS
<...SpriteNode...

``` 

# ui.Rect inconsistent
## inset
for Rect with negative width/height, most methods work normally, and min/max is computed correctly.  Most methods seem to standardize (for instance translate(0,0), or intersection with self results in a standardized rect.  inset works "backwards" for negative width/height, producing a larger rather than a smaller Rect.

```python
>>> r=ui.Rect(0,0,-100,100)
>>> abs(r.inset(0,10).width)
80.0

``` 

## ==
 also, == does not work as might be expected (ala CGRectEqualsRect) for negative width/height

```python
>>> r=ui.Rect(0,0,100,100)
>>> r==r.translate(0,0) # works
True

>>> r=ui.Rect(0,0,-100,100)
>>> r==r.translate(0,0) #fails
True

``` 


# other non-doctestable:

  * image_quad from arguments are incorrectly scaled. (workaround seems to be to multiply by 2, though i suspect this may depend on device)[not tested yet in 160037]
  * ui editor does not save the currently editing view when switching focus to console (unlike script editor, which does).   
		example:  Create new ui in ui editor. Click add, and add a button.  Switch to console, and read the pyui...the button will not be shown.     Switching to console ought to force write of the current pyui file.

* poorly formed pyui files crash pythonista.  editor.make_new_file('this_will_crash.pyui') crashes
* syntax highlighting for multiple same line imports (`import ui, os`) only highlights first module
* IndentationErrors in the traceback viewer do not navigate to, or  highlight the offending line, despite listing the line number in the traceback.
		example:
			editor.make_new_file('testme.py','''def a():\n   print 'hi'\n print 'there'\n'''

 
