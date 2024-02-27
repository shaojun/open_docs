# Manually get parameters of wechat public account 
1. regist 个人公众号  
  在这里进行注册, 要认证等https://mp.weixin.qq.com/
2. 登陆进入公众号并新创编写一篇草稿
   ![image](https://github.com/shaojun/open_docs/assets/3241829/db98c341-3e5b-4cd7-8262-2024aae1aa23)

3. 新建超链接  
   ![image](https://github.com/shaojun/open_docs/assets/3241829/d8a09a3d-9172-434f-98a8-9783909edec2)

   选择公众号搜索:  
   ![image](https://github.com/shaojun/open_docs/assets/3241829/1e9f8124-9813-4253-b1aa-80fdea400eaa)

   **然后请马上浏览器F12**
4. 搜索目标公众号 
  ![image](https://github.com/shaojun/open_docs/assets/3241829/e79bdccd-41b9-40c2-9c47-0fd151215b6e)
   下方将出现搜索列表, 请选择  
  ![image](https://github.com/shaojun/open_docs/assets/3241829/98cf6c74-7c74-4225-92f8-05659a28a450)

5. 获取关键参数  
   在浏览器F12中, 类似`appmsgpublish?sub=list&search_field=null&begin=0&coun....`这种的url就是我们要查看的对象 :  
    
  ![image](https://github.com/shaojun/open_docs/assets/3241829/6bc5c260-f1a2-4331-b936-cd219e3f2ccc)


  上方的`fakeid`即目标公众号的id, `token`也是重要参数.  
  
  ![image](https://github.com/shaojun/open_docs/assets/3241829/8fbc31c7-9688-4b6c-8f46-d5156026d0f9)


  上方的`cookie`和`User-Agent`都是是重要参数.



