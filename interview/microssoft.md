设为读从库的话，如何解决主从延迟的问题。如果用写缓存然后读缓存的方式解决的话，是先写数据库还是先写缓存？如果先写缓存的话，如果系统崩溃了，缓存写入了，但是数据库没写入，怎么办？



算法题：

/*
Q1:  

reverse string 

input: "hello  world !  "

output: "  ! world  hello"

string ReverseString(string input)

*/