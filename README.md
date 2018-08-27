# bauduindex
百度指数
## 分为模拟登陆和图像截取并识别两部分

## 需要安装的库:
    time
    selenium
    PIL
    pytesseract
    getpass
    chromedriver.exe



## 登陆部分:
    def login(browser, userName, password):
    # 输入 url
    loginUrl = "https://passport.baidu.com/v2/?login&tpl=mn&u=http%3A%2F%2Fwww.baidu.com%2F"
    # 跳转到登陆界面
    browser.get(loginUrl)
    # 超时限制
    timeout = 120 # seconds
    time.sleep(10)
    try:
        # 等待准备点击
        # 跳过扫码点击用户名登陆
        switchLoginMethod = WebDriverWait(browser, timeout).until(EC.element_to_be_clickable((By.ID, 'TANGRAM__PSP_3__footerULoginBtn')))
        switchLoginMethod.click()
        # 等待输入字段就绪
        userNameField = WebDriverWait(browser, timeout).until(EC.presence_of_element_located((By.ID, 'TANGRAM__PSP_3__userName')))
        userNameField.clear()
        userNameField.send_keys(userName)
        passwordField = WebDriverWait(browser, timeout).until(EC.presence_of_element_located((By.ID, 'TANGRAM__PSP_3__password')))
        passwordField.clear()
        passwordField.send_keys(password)
        # 提交
        submitButton = WebDriverWait(browser, timeout).until(EC.element_to_be_clickable((By.ID, 'TANGRAM__PSP_3__submit')))
        submitButton.click()


    except TimeoutException:
        print ("Loading took too much time!")
### 登陆的页面：
        # 本地输入不需要输入验证码
        ![image](http://github.com/einshan/login.png)
        
## 爬取百度指数部分：

### 打开百度指数页面并输入关键词:
    def getindex(browser, keyword, day):
    # 设置url和超时限制
    indexurl = "http://index.baidu.com"
    indexTimeout = 60
    # 进入百度指数
    browser.get(indexurl)
    # 等待页面准备就绪
    searchField = WebDriverWait(browser, indexTimeout).until(EC.presence_of_element_located((By.CSS_SELECTOR, 'input.search-input')))
    # 清空字段并输入关键词
    searchField.clear()
    searchField.send_keys(keyword)
    # 找到并点击按钮
    button = WebDriverWait(browser, indexTimeout).until(EC.element_to_be_clickable((By.CSS_SELECTOR, 'div.search-input-operate')))
    button.click()

### 找到日期：
    sel = '//a[@rel="' + str(day) + '"]'
    dayButton = WebDriverWait(browser, indexTimeout).until(EC.element_to_be_clickable((By.XPATH, sel)))
    dayButton.click()
### 设置输入天数规则：
    if day <= 7:
        day = 7
    elif day <= 30:
        day = 30
    elif day <= 90:
        day = 90
    elif day <= 180:
        day = 180
    else:
        day = "all"
### 放大屏幕：
    browser.maximize_window()
### 获取和设置图形框：
    xoyelementReady = False
    while  xoyelementReady == False:
        try:
            xoyelement = browser.find_elements_by_css_selector("#trend rect")[2]
            xoyelementReady = True
        except:
            time.sleep(2)
            print('xoyelement not ready yet')
    num = 0
    x_0 = 3
    y_0 = 0
    index = []
    # 获取图形框宽度
    xoyelement_length = xoyelement.size['width']
    if day == "all":
        day = 1000000
    ![image](http://github.com/einshan/xoyelementReady.png)
 ### 用selenium库来模拟鼠标滑动悬浮：
    ActionChains(browser).move_to_element_with_offset(xoyelement, 250, y_0).perform()
    time.sleep(2)
    moveTimeout = 5
    try:
        # 按照天数进行循环
        for i in range(day):
            # 将光标移动到下一点
            ActionChains(browser).move_to_element_with_offset(xoyelement, x_0, y_0).perform()
            # 构造规则
            if day == 7:
                x_0 = x_0 + (xoyelement_length-10)/6
            elif day == 30:
                x_0 = x_0 + (xoyelement_length-10)/29
            elif day == 90:
                x_0 = x_0 + (xoyelement_length-10)/89
            elif day == 180:
                x_0 = x_0 + (xoyelement_length-10)/179
            elif day == 1000000:
                x_0 = x_0 + 3.37222222
 ### 获取屏幕大小：
            screen = browser.find_element_by_xpath('//html')
            screenSize = screen.size

            time.sleep(2)  
 ### 截图：
            #截图范围    
            rangle = (
            int(locations['x']), int(top),
            int(locations['x'] + sizes['width'] + 1), int(top + sizes['height'] + 1))
            # 保存屏幕截图
            path = "./" + str(num)
            browser.save_screenshot(str(path) + ".png")
            img = Image.open(str(path) + ".png")
            img = img.resize((screenSize['width'], screenSize['height']))
            img.show()
            jpg = img.crop(rangle)
            jpg = jpg.convert('RGB')
            jpg.show()
            jpg.save(str(path) + ".jpg")  
             # 放大图片
            jpgzoom = Image.open(str(path) + ".jpg")
            (x, y) = jpgzoom.size
            x_s = x*2
            y_s = y*2
            out = jpgzoom.resize((x_s, y_s), Image.ANTIALIAS)
            # 保存
            out.save(path + 'zoom.jpg', 'png', quality=95)
### 图片识别：
            try:
                    image = Image.open(str(path) + "zoom.jpg")
                    code = pytesseract.image_to_string(image)
                    if code:
                        index.append(code)
                    else:
                        index.append("")
                except Exception as err:
                    print(err)
                    index.append("")
                num = num + 1
                
        
        
