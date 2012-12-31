## UTILS.JSON ##

    /// Authors: Jaume Ferre and Cristian Martin <br/>
    /// Version: 0.1 <br/>
    /// Based on Services JSON PHP class by Michal Migurski <a href="http://pear.php.net/pepr/pepr-proposal-show.php?id=198">http://pear.php.net/pepr/pepr-proposal-show.php?id=198</a><br />
    /// License: <a href="http://www.opensource.org/licenses/bsd-license.php">http://www.opensource.org/licenses/bsd-license.php</a>
    Class UTILS.JSON Extends %RegisteredObject
    {
    
    Parameter JSonSlice = 1;
    
    Parameter JSonInString = 2;
    
    Parameter JSonInArray = 3;
    
    Parameter JSonInObject = 4;
    
    ClassMethod Test(cad As %String = "") As %String
    {
        i cad="" {
            s cad="{'a':['a','b','c'],'b':2,'c':'caracter','d':{'d1':['a','b','c'],'d2':['d','e','f']},'e':4}"
            ;s cad="{'a':'A','b':'B','c':'o\'hara','d\'prima':'D'}"
            ;s cad="[1,2,3]"
            ;s cad="1"
        }
        q ..Decode(cad)
    }
    
    ClassMethod SetAux(what As %String, where As %Integer, delim As %String) As %DataType [ Internal ]
    {
        s aux=##class(%ArrayOfDataTypes).%New()	
        d aux.SetAt(what,"what")
        d aux.SetAt(where,"where")	
        d aux.SetAt(delim,"delim")
        
        q aux
    }
    
    /// we know that it's not escaped becase there is _not_ an
    /// odd number of backslashes at the end of the string so far
    ClassMethod isEscaped(str As %String, c As %String) As %Boolean [ Internal ]
    {
        s pos=$F(str,c)
        q ($L($E(str,1,pos))-$L($REPLACE($E(str,1,pos),"\","")))#2=1
    }
    
    /// scapes the string. TODO: add more scapes
    ClassMethod Escape(str As %String) As %String [ Internal ]
    {
        q $REPLACE(str,"'","\'")
    }
    
    /// Decode a string JSON. 
    ClassMethod Decode(str As %String) As %ArrayOfDataTypes
    {
        #dim stack as %ListOfDataTypes
        s matchType=$ZCVT(str,"L")
        
        q:(matchType="true") "1"
        q:(matchType="false") "0"
        q:(matchType="null") ""	
        q:($ISVALIDNUM(matchType)) matchType 
        q:str?1"'".E1"'" $replace($e(str,2,$l(str)-1),"\'","'")
        
        // array or object notation
        s match=str?1(1"[".E1"]",1"{".E1"}")
        s stack=##class(%ListOfDataTypes).%New()
        
        if match {
            if $E(str,1)="[" {
                d stack.Insert(..#JSonInArray)
                s arr=##class(%ListOfDataTypes).%New()
            }	
            else {
                d stack.Insert(..#JSonInObject)	
                s obj=##class(%ArrayOfDataTypes).%New()
            }
            
            d stack.Insert(..SetAux(..#JSonSlice,1,"false"))
            
            s chars=$E(str,2,$L(str)-1)
            
            if chars="" {
                if stack.GetAt(1)=..#JSonInArray {
                    q arr
                }
                else {
                    q obj
                }	
            }
    
            s strlenChars=$L(chars)+1
            
            For c=1:1:strlenChars {
                s last=stack.Count()
                s top=stack.GetAt(last)
                
                s substrC2=$E(chars,c-1,c)
                if $e(chars,c)="" {
                    s a=22
                }
                
                if (c=strlenChars || ($E(chars,c)=",")) && (top.GetAt("what")=..#JSonSlice) {
                    // found a comma that is not inside a string, array, etc.,
                    // OR we've reached the end of the character list
                    
                    s slice = $E(chars, top.GetAt("where"),c-1)
                    
                    d stack.Insert(..SetAux(..#JSonSlice,c+1,"false"))
                    
                    if stack.GetAt(1)=..#JSonInArray {
                        // we are in an array, so just push an element onto the stack
                        d arr.Insert(..Decode(slice))	  	
                    }
                    elseif stack.GetAt(1)=..#JSonInObject {
                        // we are in an object, so figure
                        // out the property name and set an
                        // element in an associative array,
                        // for now					  	
                        
                        s match=slice?." "1"'"1.E1"'"." "1":"1.E
                        if match {
                            //'name':value par
                            
                            ;s slice=$REPLACE(slice," ","")
                            s key1=$p(slice,":")
                            s key=..Decode(key1)
                            
                            s val=..Decode($P(slice,":",2,$l(slice,":")))	
                            d obj.SetAt(val, key)
                            
                        }
                    }
                }
                elseif $E(chars,c)="'" && (top.GetAt("what")'=..#JSonInString) {
                    // found a quote, and we are not inside a string
                    d stack.Insert(..SetAux(..#JSonInString,c,$E(chars,c)))
                }
                elseif $E(chars,c)=top.GetAt("delim") && (top.GetAt("what")=..#JSonInString) && (substrC2'="\'") {
                    // found a quote, we're in a string, and it's not escaped
                    s last=stack.Count()
                    s st=stack.RemoveAt(last)
                }
                elseif $E(chars,c)="[" && ($CASE(top.GetAt("what"),..#JSonInString:1,..#JSonInArray:1,..#JSonSlice:1,:0)) {
                    // found a left-bracket, and we are in an array, object, or slice
                    d stack.Insert(..SetAux(..#JSonInArray,c,"false"))
                }
                elseif $E(chars,c)="]" && (top.GetAt("what")=..#JSonInArray) {
                    // found a right-bracket, and we're in an array
                    s last=stack.Count()
                    s st=stack.RemoveAt(last)	
                }
                ;modificacio 19/11/08: ..#JSonString -> #JSonInArray
                elseif $E(chars,c)="{" && ($CASE(top.GetAt("what"),..#JSonSlice:1,..#JSonInArray:1,..#JSonInObject:1,:0)) {
                    // found a left-brace, and we are in an array, object, or slice
                    d stack.Insert(..SetAux(..#JSonInObject,c,"false"))
                }
                elseif $E(chars,c)="}" && (top.GetAt("what")=..#JSonInObject) {
                    // found a right-brace, and we're in an object	
                    s last=stack.Count()
                    s st=stack.RemoveAt(last)	
                }	
                
            }	
            
            if stack.GetAt(1)=..#JSonInObject {
                q obj
            }
            elseif stack.GetAt(1)=..#JSonInArray {
                q arr
            }
        }
        q str
    }
    
    /// Encode a Cache string to a JSON string
    ClassMethod Encode(data As %DataType) As %String
    {
    
        if $IsObject(data) {	
            s key=""
            
            s typeData=data.%ClassName()
    
            if typeData="%ArrayOfDataTypes" || (typeData = "ArrayOfDT") {
                //type object
                s key=""
                s cad=""
                F {
                    s pData=data.GetNext(.key)
                    q:key=""
                    s value=..Encode(pData)
                    s cad=$S(cad'="":cad_",",1:"")_"'"_..Escape(key)_"':"_value	
                } 
                q "{"_cad_"}"
            }
            elseif typeData="%ListOfDataTypes" || (typeData = "ListOfDT") {
                //type array	
                
                s cad=""
                f i=1:1:data.Count() {
                    s tmp=..Encode(data.GetAt(i))
                    s cad=$S(i>1:cad_",",1:"")_tmp
                }
                
                s cad="["_cad_"]"
                q cad
            }
        }
        elseif $ISVALIDNUM(data) {
            // type number
            q data
        }
        else {
            //type string
            q:data="" "null"
            q "'"_..Escape(data)_"'"
        }
    }
    
    }
