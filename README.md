# Widget Extensions

Widget extensions are similar to other extensions in that they are zip packages that can be imported into the Thingworx platform and that they can be created using the Eclipse plugin. Unlike other extensions, they are created using client-side HTML, CSS, and Javascript. Creating a new widget for ThingWorx is incredibly straightforward and powerful. Any capability available from JavaScript and HTML can be encapsulated and created as a widget within ThingWorx. Every widget in ThingWorx’s Mashup environment was built with the same SDK we deliver to you.

## Hello World Widget

For the purposes of demonstration, lets create a simple "Hello World" Widget using the Eclipse plugin. First, create a new Thingworx Extension Project using File > New > Thingworx Extension Project. Choose a project name, select you SDK location, and enter the vendor information and select Finish:

![](WidgetAPI.png)

Now, right click out project and select New > Widget. Enter Hello Widget for our widget's name and select Finish
![](WidgetAPI-1.png)

You should have a project structure that looks like this:
![](WidgetAPI-2.png)

### Metadata.xml
As with other extension projects, there will be a metadata file to explain our extension to the platform. Let's open the file metadata.xml under the config files folder

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<Entities>
    <ExtensionPackages>
        <ExtensionPackage dependsOn="" description="" minimumThingWorxVersion="8.0.0" name="HelloWorld" packageVersion="1.0.0" vendor="Thingworx Labs"> 
        </ExtensionPackage>
    </ExtensionPackages>
    <Widgets>
        <Widget name="hello_world">
            <UIResources>
                <FileResource description="" file="hello_world.ide.js" isDevelopment="true" isRuntime="false" type="JS"/>
                <FileResource description="" file="hello_world.ide.css" isDevelopment="true" isRuntime="false" type="CSS"/>
                <FileResource description="" file="hello_world.runtime.js" isDevelopment="false" isRuntime="true" type="JS"/>
                <FileResource description="" file="hello_world.runtime.css" isDevelopment="false" isRuntime="true" type="CSS"/>
            </UIResources>
        </Widget>
    </Widgets>
</Entities>

```

Here we can see a few important details, such as our extension package name, minimum thingworx version, version, and vendor. We also see a description of the widgets included in this extension package under the Widgets tag. This is where we name our widget and include any file resources that our widget will use. 

Let's look at each of these in turn

#### hello_world.ide.js and hello_world.ide.css

One thing that you might notice about widgets in Thingworx is that they work completely differently in the mashup builder than they do in the runtime webpage environment. This means you have to describe how your widget will work both in the IDE, or Mashup Builder, and how it will work in the Runtime environment. The ide.js file describes how your widget will behave in the mashup builder. Similarly, hello_world.ide.css describes how your widget will be _styled_ in the mashup builder. **Note  that both of these files are flagged as "isDevelopment"="true" and "isRuntime"="false" **. These flags are incredibly important because they tell the platform whether or not to include the file in the combined javascript and combined CSS that is built for the Mashup Builder and Runtime environment. In this case, our ide.js and ide.css files will be included in the Mashup Builder, but not in the Runtime environment -- which makes sense!

Also note that the type element is set to either "JS" or "CSS", to match the type of file resource that you are including

#### hello_world.runtime.js and hello_world.ide.css

Similar to the ide.js and ide.css files, these files describe how your widget will behave in the runtime environment. Note that these files are flagged as "isDevelopment"="false" and "isRuntime"="true", because in this case we want them included in the combined extension javascript and css in the Runtime environment, rather than the Mashup Builder.

#### Other file resources

You can also reference additional file resources that you want to include with your package here, simply by adding another FileResource line. For example, you may wish to include a third-party javascript or css library, or a font resource. These files should be placed in the ui>widget_name folder, or referenced with a relative path from the ui>widget_name folder. File resources with types other than Javascript or CSS should be included with the type of "UNKNOWN". For example, if I wanted to include some additional fonts in my widget, I might create a fold "fonts" in /ui/hello_world and add the line

```xml
 <FileResource description="" file="fonts/myfont.ttf" isDevelopment="true" isRuntime="true" type="UNKNOWN"/>
```
Note that here we may want to include this file in both the Mashup Builder and in the Runtime environment and in this case have flagged them both as true.

#### A note on third-party libraries

One final thing to consider before including third-party libraries is that Thingworx offers a number of common Javascript libraries as part of the platform. These libraries do not need to be included as part of your widget and, in fact, could create conflicts with existing libraries on the Thingworx Platform. You can see which libraries are included in the Tomcat /webapps/Thingworx/Common folder on the server where Thingworx is installed

### The IDE JS File -- a deeper dive

Now that we have looked at the metadata structure of our widget, lets dive into the actual code that comprises our widget.

The ide.js has a few simple methods you must implement. The widgets are declarative. There are 3 functions where you declare your widget’s properties, services and events. Additionally, there are a number of callback functions that you may implement which will be called as part of the widget lifecycle. 

In our Hello World widget, we see the following functions defined:

* widgetIconUrl - returns the url of the icon we want to display in the widget list of the Mashup builder
* widgetProperties - returns a JSON structure defining the properties of this widget
* afterSetProperty - called after any property is updated within the Composer, return true to have the properties re-rendered in composer
* renderHtml - returns HTML fragment that the Composer will place in the screen; the widget’s content container (e.g. div) must have a ‘widget-content’ class specified, after this container element is appended to the DOM, it becomes accessible via jqElement and its DOM element id will be available in jqElementId
* afterRender - called after we insert your html fragment into the dom


### The Runtime JS File -- A Deeper Dive

Similar to the IDE JS file, the Runtime JS file has a few simple methods we must implement. In our hello world example we have the following:

* renderHtml - return any HTML you want rendered for your widget
* afterRender - called after we insert your html fragment into the dom
* updateProperty - this is called on your widget anytime bound data changes. For each data binding, your widget’s updateProperty() will be called each time the source data is changed. You need to check updatePropertyInfo.TargetProperty to determine what aspect of your widget should be updated.

##### Binding widget events

```javascript
thisWidget.jqElement.bind('dblclick', function (e) {
  thisWidget.jqElement.triggerHandler('DoubleClicked');
  e.preventDefault();
});
```

#### Handling service invokation
serviceInvoked() is called whenever a service you defined in the IDE JS file is triggered.

```javascript
this.serviceInvoked = function (serviceName) {
  var widgetReference = this;
      if (serviceName === 'ServiceName') {
            //handle invoked service
      } else {
        TW.log.error('Navigation widget, unexpected serviceName invoked "' + serviceName + '"');
      }
  };
```


##### InfoTable Properties

Unlike other widget properties, you cannot access InfoTable properties using properties['propertyName']. InfoTable objects are only available in the updateProperty callback by using updatePropertyInfo.ActualDataRows

###### Handling updateProperty

For each data binding, your widget’s updateProperty() will be called each time the source data is changed. You need to check updatePropertyInfo.TargetProperty to determine what aspect of your widget should be updated

In our simple example, we have a single text property defined in our IDE, and updating it looks like this:

```javascript
	this.updateProperty = function (updatePropertyInfo) {
		// TargetProperty tells you which of your bound properties changed
		if (updatePropertyInfo.TargetProperty === 'Hello World Property') {
			valueElem.text(updatePropertyInfo.SinglePropertyValue);
			this.setProperty('Hello World Property', updatePropertyInfo.SinglePropertyValue);
		}
	};
```
Here we are checking to see if the updated property that has changed is our Hello World Property and if so we are updating our valueElem.text value. Additionally, we are setting our widget's property to our new value -- **Note:** if you do not call this.setProperty here, the property value in the widget's properties object will remain the same, so even though you may have handled the update with the new value, if you  reference it again with the properties object elsewhere in the code you will get the old value.

Updating an infotable property, similarly, would check to see that TargetProperty and then handle the update using the ActualDataRows element of the updatePropertyInfo object:

```javascript
	this.updateProperty = function (updatePropertyInfo) {
		// TargetProperty tells you which of your bound properties changed
		if (updatePropertyInfo.TargetProperty === 'Hello World Property') {
			valueElem.text(updatePropertyInfo.SinglePropertyValue);
			this.setProperty('Hello World Property', updatePropertyInfo.SinglePropertyValue);
		}
		
		if (updatePropertyInfo.TargetProperty === "Data") {
			var rows = updatePropertyInfo.ActualDataRows
			
			for (i=0;i<rows.length;i++) {
				var row = rows[i];
				
				//here is where the magic happens
				
			}
		}
	};
```

###### Handling handleSelectionUpdate
If your widget allows for selection of a row in an InfoTable, it must implement the handleSelectionUpdate callback. This is called whenever selectedRows has been modified by the data source you’re bound to on that PropertyName. selectedRows is an array of the actual data and selectedRowIndices is an array of the indices of the selected rows

```javascript
   	this.handleSelectionUpdate = function (propertyName, selectedRows, selectedRowIndices) {
     	//update your widget to reflect the new selected value
   	}
```

###### Handling updateSelection

This is the counter part to handleSelectionUpdate. Your widget must call this function any time a row is selected to notify the InfoTable on the bound property that the SelectedRows has changed. **Note: this function will not work correctly if the handleSelectionUpdate callback function is not defined**

An example might be a handleClick function that we have bound to a click event on our widget:

```javascript
this.handleClick = function(evt)
   	{
   	  //Here you need to figure out what row index was selected at the click event of your widget. 
   	  var selectedRows = [];
   	  
   	  //for each row that is selected on your widget, add the index of that row to your selectedRows array
   	  
   	  selectedRows.push(mySelectedRowIndex)
   	  thisWidget.updateSelection('Data', selectedRows);
   	}
```

##### Styles and State Based Formatting

If you have a property with baseType of STYLEDEFINITION, you can get the style information by calling 
```javascript
var formatResult = TW.getStyleFromStyleDefinition(this.properties['PropertyName']);
```
If you have a property of baseType of STATEFORMATTING

```javascript 
var formatResult = TW.getStyleFromStateFormatting({
DataRow: row,
StateFormatting: thisWidget.properties['PropertyName']
});
```

Where row is the full row object that you wish to get the style definition for

In both cases formatResult is an object with the following defaults:
```javascript 
{
  image: '',
  backgroundColor: '#000000',
  foregroundColor: '#000000',
  fontEmphasisBold: false,
  fontEmphasisItalic: false,
  fontEmphasisUnderline: false,
  displayString: '',
  lineThickness: 1,
  lineStyle: 'solid',
  lineColor: '',
  secondaryBackgroundColor: '',
  textSize: 'normal'
}
```

You can get the actual font size from the textSize attribute by calling 

```javascript
var style = TW.getStyleFromStyleDefinition(this.properties['PropertyName']);
TW.getTextSize(style.textSize)
```
which will return a string in the format of "font-size: 12px;"

##### Additional Utility Functions

##### Tips and Tricks
Use this.jqElement to limit your element selections, this will reduce the chance of introducing unwanted behaviors in the application when there might be duplicate IDs and/or classes in the DOM.

* Don’t do...
    * $(‘.add-btn’).click(function(e) { ...do something... });
* Do...
    * this.jqElement.find(‘.add-btn’).click(function(e) { ...do something... });

Logging - we recommend that you use the following methods to log in the Widget Composer and Runtime environment:

* TW.log.trace(message[, message2, ... ][, exception])
* TW.log.debug(message[, message2, ... ][, exception])
* TW.log.info(message[, message2, ... ][, exception])
* TW.log.warn(message[, message2, ... ][, exception])
* TW.log.error(message[, message2, ... ][, exception])
* TW.log.fatal(message[, message2, ... ][, exception])

You can view the log messages in the Mashup Composer by opening the log window via the Help>Log menu item; in the mashup runtime, you can now click on the "Show Log" button on the top left corner of the page to show log window. If the browser you use supports console.log(), then the messages will also appear in the debugger console.



#### widgetProperties [required]

**[comment]: this should probably be a seperate page**

This function returns, essentially, the JSON structure that makes up your widget description in the IDE. The only required property is the "name" field, and there are additionally the following optional properties:

* description - a description of the widget; used for tooltip
* category - an array of strings for specifying one or more categories that the widget belongs to (i.e. Common, Charts, Data, Containers, Components); enables the user to filter widgets by type/category
* isResizable - true or false (default to true)
* supportsAutoResize - true or false (default to false) - whether this widget is responsive
* defaultBindingTargetProperty - name of the property to use as data/event binding target
* borderWidth - if your widget provides a border, set this to the width of the border. This helps ensure pixel-perfect WYSIWG between builder and runtime.
If you set a border of 1px on the “widget-content” element at design time, you are effectively making that widget 2px taller and 2px wider (1px to each side). To account for this descrepancy, setting the borderWidth property will make the design-time widget the exact same number of pixels smaller. Effectively, this places the border “inside” the widget that you have created and making the width & height in the widget properties accurate.
* isContainer - true or false (default to false); controls whether an instance of this widget can be a container for other widget instances
* customEditor - name of the custom editor dialog to use for entering/editing the widget’s configuration. If you put xxx, the system presumes you have created TW.IDE.Dialogs.xxx that conforms to the Mashup Widget Custom Dialog API (described in a separate document). We could support the ability to specify an array here as well where each entry would create an additional tab in the widget configuration dialog. For 1.1 we’ll limit this to a string and only one custom configuration.
* customEditorMenuText: the text that will appear on the flyout menu for your widget as well as the hover text over the configure widget properties button. For example: 'Configure Grid Columns'.
* allowPositioning - optional, true or false (default to true)
* supportsLabel - optional, true or false (default to false); if true, the widget will expose a ‘Label’ property whose value will be used to create a text label that sits next to the widget in the Composer and runtime
* supportsAutoResize - optional, true or false (default to false); if true, the widget will be able to be placed directly into responsive containers (columns, rows, responsive tabs, responsive mashups, etc.
* properties - a collection of property (attribute) objects; each property object can have...
    * property name :
    * description - a description of the widget; used for tooltip
    * baseType- the system base type name; in addition, if the baseType value is ‘FIELDNAME’, the widget property window will display a dropdown list that allows the user to pick from a list of fields available in the INFOTABLE bound to the sourcePropertyName value, based on the baseTypeRestriction specified; for an example, see the TagCloud widget implementation. Other special baseTypes:
        * STATEDEFINITION just picks a StateDefinition
        * STYLEDEFINITION just picks a StyleDefinition
        * RENDERERWITHSTATE will show a dialog and allow you to select a renderer and formatting that goes along with it. Note: you can set a default Style by putting the string with the default style name in the ‘defaultValue’. Also, note that anytime your binding changes, you should reset this to the default value as in the code below:
        ```javascript 
        this.afterAddBindingSource = function (bindingInfo) {
          if (bindingInfo['targetProperty'] === 'Data') {
          this.resetPropertyToDefaultValue('ValueFormat');
          }
        };
        ```
       * STATEFORMATTING will show a dialog and allow you to pick either fixed style or a state-based style. Note: you can set a default Style by putting the string with the default style name in the ‘defaultValue’. Also, note that anytime your binding changes, you should reset this to the default value as in the code above for RENDERERWITHSTATE.
        * VOCABULARYNAME will just pick a DataTags vocabulary at the moment
   * mustImplement - if the baseType is THINGNAME, and you specify “mustImplement” the Composer will restrict to popups implementing the specified EntityType and EntityName [by calling QueryImplementingThings against said EntityType and EntityName] 
   ```javascript
     'baseType': 'THINGNAME',
     'mustImplement' : {
       'EntityType' : 'ThingShapes',
       'EntityName' : 'Blog'
      },
    ```
    * baseTypeInfotableProperty - if baseType is RENDERERWITHFORMAT, baseTypeInfotableProperty specifies which property’s infotable is used for configuration
    * sourcePropertyName - when the property’s baseType is ‘FIELDNAME’, this attribute is used to determine which INFOTABLE’s fields are to be used to populate the FIELDNAME dropdown list; for an example, see the TagCloud widget implementation
    * baseTypeRestriction - when specified, this value is used to restrict the fields available in the FIELDNAME dropdown list; for an example, see the TagCloud widget implementation
    * tagType - if the baseType is ‘TAGS’ this can be ‘DataTags’ or ‘ModelTags’ … defaults to DataTags
    * defaultValue - default undefined; used only for ‘property’ type
    * isBindingSource - true or false; allows the property to be a data binding source, default to false
    * isBindingTarget - true or false; allows the property to be a data binding target, default to false
    * isEditable - true or false; controls whether the property can be edited in the Composer, default to true
    * isVisible - true or false; controls whether the property is visible in the properties window, default to true
    * isLocalizable - true or false; only important if baseType is ‘STRING’ - controls whether the property can be localized or not.
    * selectOptions - an array of value / (display) text structures
      Example: [ { value: ‘optionValue1’, text: ‘optionText1’}, { value: ‘optionValue2’, text: ‘optionText2’} ]
    * warnIfNotBoundAsSource - true or false; if true, then the property will be checked by the Composer for whether it’s bound and generate a to-do item when it’s not
    * warnIfNotBoundAsTarget - true or false; if true, then the property will be checked by the Composer for whether it’s bound and generate a to-do item when it’s not
    
    #### Custom Editors
    
    **[comment] Need to include some details on creating custom editors here**


#### Widget Events [optional]

A collection of events; each event can have
```javascript
  this.widgetEvents = function () {
    return {
      'DoubleClicked': {}
    };
 };
 ```
 
 #### Widget Services [optional]
 
 A collection of services; each service can have
    
```javascript
this.widgetServices = function () {
    return {
        'ResetZoom': { 'warnIfNotBound': false }
    };
};
```

### API Provided for your Widget - IDE

Calls/properties that runtime provides for a widget
* this.jqElementId
    * this is the DOM element ID of your object after renderHtml
* this.jqElement
   * this is the jquery element
* this.getProperty(name)
* this.setProperty(name,value)
    * Note: in the IDE every call to this will call afterSetProperty() if it’s defined in your widget
* this.updatedProperties()
    * If you change the properties of your object at any point, please call this.updatedProperties() to let the Builder know that it needs to update the widget properties window, the connections window, etc.
* this.getInfotableMetadataForProperty(propertyName)
    * if you need the infotable metadata for a property you have bound, you can get it by calling this API … it returns undefined if you’re not bound.
* this.resetPropertyToDefaultValue(propertyName)
    * this resets the named property back to whatever it’s default value is.
* this.removeBindingsFromPropertyAsTarget(propertyName)
    * this removes any target data bindings from this propertyName … use this only when the user has initiated an action that invalidates that property.
* this.removeBindingsFromPropertyAsSource(propertyName)
    * this removes any source data bindings from this propertyName … use this only when the user has initiated an action that invalidates that property.
* this.isPropertyBoundAsTarget(propertyName)
    *  this returns whether the property has been bound as a target. Most useful when trying to validate if a property has been either set or bound
* this.isPropertyBoundAsSource(propertyName)
    *  this returns whether the property has been bound as a source. Most useful when trying to validate if a property has been bound to a target

#### API Provided for your Widget - Runtime
Calls/properties that runtime provides for a widget
* this.domElementId
    * this is the DOM element ID of your object after renderHtml
* this.jqElement
    * this is the jquery element
* this.getProperty(name)
* this.setProperty(name,value)
* this.updateSelection(propertyName,selectedRowIndices)
    * call this anytime your widget changes selected rows on data bound to a certain propertyName … e.g. in a callback you have for an event like onSelectStateChanged you’d call this API and the system will update any other widgets relying on selected rows

#### Widget Lifecycle in the Composer
* discovered (because of js being loaded into Index.html) and added to the widget toolbar/palette
    * widgetProperties is called to get information about each widget (e.g. display name and description)
    * widgetEvents is called to get information about the events each widget exposes
    * widgetServices is called to get information about the serviceseach widget exposes
* created (e.g. dragged onto a mashup panel)
    * afterLoad is called after your object is loaded and properties have been restored from the file, but before your object has been rendered
* appended to the workspace DOM element
    * renderHtml is called to get a HTML fragment that will get inserted into the mashup DOM element
    * afterRender is called after the HTML fragment representing the widget has been inserted into the mashup DOM element and a usable element ID has been assigned to the DOM element holding the widget content, the DOM element is now ready to be manipulated
* updated (e.g. resized, or updated via the widget property window)
    * beforeSetProperty is called before any property is updated
    * afterSetProperty is called after any property is updated
* destroyed (i.e. when it’s deleted from the mashup)
    * beforeDestroy is called right before the widget’s DOM element gets removed and the widget is detached from its parent widget and dellocated; this is the place to perform any clean-up of resources (e.g. plugins, event handlers) acquired throughout the lifetime of the widget

#### Additional Callbacks from Composer to your Widget
* afterRender() [optional]
    * called after we insert your html fragment into the dom
* beforeDestroy() [optional]
    * called right before the widget’s DOM element gets removed and the widget is detached from its parent widget and dellocated; this is the place to perform any clean-up of resources (e.g. plugins, event handlers) acquired throughout the lifetime of the widget
* beforeSetProperty(name,value) [optional] [Composer only - not at runtime]
    * called before any property is updated within the Composer, this is a good place to perform any validation on the new property value before it is committed;
    * if a message string is returned, then the message will be displayed to the user, and the new property value will not be committed
* afterSetProperty(name,value) [optional] [Composer only - not at runtime]
    * called after any property is updated within the Composer
    * return true to have the widget re-rendered in the Composer
* afterAddBindingSource( bindingInfo ) [optional]
    * whenever data is bound to your widget, you will called back with this (if you implement it … it’s optional)
    * The only field in bindingInfo is targetProperty which is the propertyName that was just bound
* validate() [optional]
    * called when the Composer refreshes its to-do list;
    * the call must return an array of result object with severity (optional and not implemented) and message (required) properties;
    * the message text may contain one or more pre-defined tokens, such as {target-id}, which will get replaced with a hyperlink that allows the user to navigate/select the specific widget that generated the message

