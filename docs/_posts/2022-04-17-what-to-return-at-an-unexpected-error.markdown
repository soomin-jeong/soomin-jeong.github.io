---
layout: post
title:  "What to Return at an Unexpected Error from API"
date:   2022-04-17 00:00:00 +0200
categories: clean-code
---

<b>TL; DR:</b> API returns something crazy. What should this API calling component return? `None`? `-1`? `""`? None of the above. Raise an exception.

<h2> Problem </h2>
<p> There is a function that gets a response from an API and returns it to another function in a specific data type. This post is about what to return when the API returns an error. </p>
<p> A few (quite common in reality) characteristics of this error: </p>
<ul>
<li> The error message from the API does not contain much detail to help developers to understand, or debug the problem.</li>
<li> The product behind the API is still under development; hence, it is likely that the error is something unexpected at all. </li>
<li> The client calling the API system requires high accuracy. It would rather break the pipeline and goes down than receive any unreliable values. </li>
</ul>

With these given, it is more reliable to fail the program when the target API returns something unexpected, instead of dealing with it based on the wrong assumptions.

<h2> Extra Requirements </h2>
To be a little more graceful, it would be great if the function logs the error message, so that the developers could have a bit more hint. the program instantly stops, without making more error messages due to the cascading errors after the unexpected response.
how the function handles the error is written in scalable architecture

<h2> Solution </h2>
1. Create an exception class (api_exception.py) away from the one calling the API
2. Add an exception message in the `__init__` so that the main Exception class can handle the logging

```
# api_exception.py
from typing import List
class ApiException(Exception):
    def __init__(self, school: str, class_num: int, attrs=List[str]):
        self.school = school
        self.class_num = class_num
        self.attrs = attrsmsg = f"API call to class_num({class_num}) at school({school}) to obtain {attrs} failed"
        
        super().__init__(msg)
```

3. Raise the Exception in the component calling the API (`client.py`) so that the program fails instantly when given an error
```
# client.pyfrom typing import Any
from typing import cast
from typing import Dict
from typing import List

class APIClient:
  
    """
    calls the third-party API with the given parameters
    """
   def call_api(school: str, class_num: int, attrs: List[str]) -> List[Dict[str, Any]]:
      
        # API is expcected to return something like this...
        dummy_res = [{"name": "Jane Doe", "age": 7}, 
                     {"name": "John Doe", "age": 8}]
        return dummy_res
    def load_from_api(
        self, school: str, class_num: int, attrs: List[str]
    ) -> List[Dict[str, Any]]:  
      
        try:
            data = call_api(school, class_nu, attrs)
            return cast(List[Dict[str, str]], data)        
        except Exception as err:
            raise ApiException(
                school=school, class_num=class_num, attrs=attrs
            ) from err
```

<h2> Don'ts </h2>
1. Leave the log simply in the API-calling component and return something unexpected

```
...
import logging
logger = logging.getLogger(__name__)
...
def load_from_api(
        self, school: str, class_num: int, attrs: List[str]
    ) -> List[Dict[str, Any]]:        
    
     try:
         data = call_api(school, class_nu, attrs)
         return cast(List[Dict[str, str]], data)        
     
     except Exception as err:
         logger.error(f"API call to class_num({class_num}) at school({school}) to obtain {attrs} failed")
         return -1    # None, or empty string
```
This code does not conform to the expected data type to return.

2. Okay, then what about we return something that conforms to the expected data type?

```
try:
      data = call_api(school, class_nu, attrs)
      return cast(List[Dict[str, str]], data)
except Exception as err:
      return cast(List[Dict[str, str]], None)
```

This could be misleading because the API may return `None`.
Takeaways
Avoid returning something arbitrary (i.e. `-1`, `None`, `""` ), but raise an exception, because returning such values can be misleading
Raise an exception that is broad enough, when a third-party API specification is not clear. It should be ready to log whatever API returns.

<h3> Resources </h3>
<ul>
<li> Intellectual hints: my colleague M. </li>
<li> Photo by Andrea Piacquadio from Pexels [Link](https://medium.com/r/?url=https%3A%2F%2Fwww.pexels.com%2Fphoto%2Fadult-surprised-executive-man-using-computer-beside-ethnic-colleague-in-open-space-office-3755700%2F) </li>
<li> Icon in the figure: Server icons created by Pixel perfect Flaticon [Link](https://medium.com/r/?url=https%3A%2F%2Fwww.flaticon.com%2Ffree-icons%2Fserver) </li> 
</ul>
