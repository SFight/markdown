# C++替换所有字符串函数

```c++
#include   <string>     
    #include   <iostream>     
    using   namespace   std;     
    string&   replace_all(string&   str,const   string&   old_value,const   string&   new_value)     
    {     
        while(true)   {     
            string::size_type   pos(0);     
            if(   (pos=str.find(old_value))!=string::npos   )     
                str.replace(pos,old_value.length(),new_value);     
            else   break;     
        }     
        return   str;     
    }     
    string&   replace_all_distinct(string&   str,const   string&   old_value,const   string&   new_value)     
    {     
        for(string::size_type   pos(0);   pos!=string::npos;   pos+=new_value.length())   {     
            if(   (pos=str.find(old_value,pos))!=string::npos   )     
                str.replace(pos,old_value.length(),new_value);     
            else   break;     
        }     
        return   str;     
    }     
    int   main()     
    {     
        cout   <<   replace_all(string("12212"),"12","21")   <<   endl;     
        cout   <<   replace_all_distinct(string("12212"),"12","21")   <<   endl;     
    }     
    /* 
    输出如下:    
    22211    
    21221 
    */
```
