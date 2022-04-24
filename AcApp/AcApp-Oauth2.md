# OAuth2流程

![oauth2](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/501/acapp-oauth2.png)

## 一、申请授权码code

- 前端

    ```js
    前端通过ajax发送请求
    acwing_login(){
        $.ajax({
            url : "https://www.lelelpx.top/settings/acwing/web/apply_code/",
            type : "GET",
            success : function(resp){
                if(resp.result === 'success'){
                    window.location.replace(resp.apply_code_url);
                }
            }
        })
    }
    ```

- 后端

    ```python
    后端接收请求, 通过API去申请Code
    1. api需要的参数: appid, redirect_uri, scope, state
    2. redirect_uri: 接收Code地址
    3. scope: 申请授权的范围
    4. state: 用于判断请求和回调的一致性，授权成功后后原样返回。该参数可用于防止csrf攻击（跨站请求伪造攻击），建议第三方带上该参数，可设置为简单的随机数
        
    def apply_code(request):  # 从acwing的api获取code
        appid = "1111"
        redirect_uri = quote("https://www.lelelpx.top/settings/acwing/web/receive_code/")
        scope = "userinfo"
        state = get_state()
        cache.set(state, True, 7200)  #有效期两小时
        apply_code_url = "https://www.acwing.com/third_party/api/oauth2/web/authorize/"
        return JsonResponse({
            'result' : 'success',
            'apply_code_url' : apply_code_url + "?appid=%s&redirect_uri=%s&scope=%s&state=%s" % (appid, redirect_uri, scope, state),
        })
    ```

## 二、申请授权令牌access_token和用户的openid

- 接收Code后

    ```python
    1. 查看state是否存在并且为True
    2. 通过API申请access_token与Acwing用户openid, 参数为code, appid,secret
    
    # 申请token部分
    data = request.GET
    code = data.get('code')
    state = data.get('state')
    if not cache.has_key(state):
        return redirect("index")
    cache.delete(state)
    
    # 使用code + appid + secret 访问acwing获取信息.. (token, openid)
    apply_access_token_url = "https://www.acwing.com/third_party/api/oauth2/access_token/"
    params = {
        'appid' : '1111',
        'secret' : 'ac8583918d1a4d80a95a3d37e42477ab',
        'code' : code,
    }
    access_token_res = requests.get(apply_access_token_url, params=params).json()
    
    ```

## 三、申请用户信息

- 使用token继续请求所需信息(用户信息)

    ```Python
    1. 判断当前openid是非存在, 存在直接从数据库取出
    2. 通过API获取用户信息, 参数为token与openid
    
    # 使用token openid继续访问acwing的api获取头像与名称
    get_userinfo_url = "https://www.acwing.com/third_party/api/meta/identity/getinfo/"
    params = {
        'access_token' : access_token,
        'openid' : openid,
    }
    userinfo_res = requests.get(get_userinfo_url,params=params).json()
    
    do...
    ```

    
