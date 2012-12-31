JSONCache
=========

This class is adapted from the JSON's PHP class made by Michal Migurski (http://pear.php.net/pepr/pepr-proposal-show.php?id=198) under GPL license.

There is two archives: json.js and JSON.cls

The json.js archive is an adaption from Json class from Mootools for encode a javascript object.

Example of use:

    Javascript Code:
    
    var ob={
      tag1:["a","b","c"],
      tag2:{
          a:1,
          b:2,
          c:["c1","c2","c3"]
        },
        tag3:"Hello",
        tag4:12.3
      }     
    var str = Json.toString(ob);    
    zenPage.MyServerMethod(str);


Server Method :

    Method MyServerMethod(str As %String) [ ZenMethod ]
    {
        s obj=##class(Utilities.JSON).Decode(str)
        s tag1=ob.GetAt("tag1")
        w tag1.GetAt(2)        // prints "b"
        s tag2=ob.GetAt("tag2")
        s a=tag2.GetAt("a")
        w a        // prints 1                
    }

And inverse:

Server Method:

    Method AnotherServerMethod() As %String [ ZenMethod ]
    {
        s a=##class(aObject).%OpenId("SomeId")
        s ob=##class(%ArrayOfDataTypes).%New()        // this works too with %ListOfDataTypes
        d ob.SetAt(a.%Id(),"id")
        d ob.SetAt(a.Name,"name")
        d ob.SetAt(a.Address,"address")
            
        q ##class(Utilities.JSON).Encode(ob)
    }


Javascript Method:

    var myObject=zenPage.AnotherServerMethod();
    var ob=Json.evaluate(myObject);
    alert(ob.id);        // alert the id
    
     