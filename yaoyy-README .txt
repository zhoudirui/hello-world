##开发环境
1. mysql-5.6.7,elasticsearch-2.2.1,redis-2.4.5(需用指定版本的软件否则会有兼容性问题)
2. 项目数据库在/doc/database/yaoyy20170505.sql(数据的备份文件)
3. tomcat8 jdk8

##启动项目前的配置
1. ms-biz/src/main/resources/config/application.yml
2. ms-boss/src/main/resources/config/application.yml
3. ms-service/main/resources/config/
  wechat,短信,数据库,elasticsearch,redis,alipay的配置
  

##商品增删改查功能的简单实现文档
1. 产出产品原型(以商品模块为例)
2. 前端和设计根据产品原型来设计页面.
3. 根据需求设计数据表结构,根据表结构写Mybatis的Mapper和Dao接口 实体类和Vo.
com.ms.dao.CommodityDao
com.ms.dao.model.Commodity
com.ms.dao.vo.CommodityVo
mapper/CommodityMapper.xml
同时还有对应的Service接口和实现类(这个不一定需要)
com.ms.service.CommodityService
com.ms.service.impl.CommodityServiceImpl
4. 等待前端完成页面的同时 后端根据业务逻辑写Controller和Server层的业务代码
例如商品模块的功能是,列表展示和增删改查类似下面Controller的代码 没有具体实现只是根据需求列出需要的接口.

`    /**
     * 商品列表页面
     * @param commodity
     * @param pageNum
     * @param pageSize
     * @return
     */
    @RequestMapping(value = "list", method = RequestMethod.GET)
    public String list(CommodityVo commodity, Integer pageNum, Integer pageSize, ModelMap model) {
    }
    /**
     * 添加商品页面
     * @return
     */
    @RequestMapping(value = "add", method = RequestMethod.GET)
    @SecurityToken(generateToken = true)
    public String add() {
        // 获取单位信息
        return "commodity_add";
    }
    /**
     * 商品详情
     * @param id
     * @param model
     * @return
     */
    @RequestMapping(value = "detail/{id}", method = RequestMethod.GET)
    @SecurityToken(generateToken = true)
    public String detail(@PathVariable("id") Integer id, ModelMap model) {
        // 获取单位信息
        CommodityVo vo = commodityService.findById(id);
        model.put("commodity", vo);
        return "commodity_editor";
    }
    /**
     * 保存
     * @param commodity
     * @return
     */
    @RequestMapping(value = "save", method = RequestMethod.POST)
    @ResponseBody
    @SecurityToken(validateToken = true)
    @BizLog(type = LogTypeConstant.COMMODITY, desc = "保存商品信息")
    public Result save(@RequestBody CommodityVo commodity) {
        return Result.success("保存成功!");
    }
    /**
     * 删除
     * @param id
     * @return
     */
    @RequestMapping(value = "detete/{id}", method = RequestMethod.POST)
    @ResponseBody
    @BizLog(type = LogTypeConstant.COMMODITY, desc = "删除商品")
    public Result delete(@PathVariable("id") Integer id) {
        commodityService.deleteById(id);
        return Result.success("删除成功!");
    }`

5. 在Service 里面写具体实现代码了然后在Controller层调用 完成后台相关逻辑.
6. 前端页面已经完成,前后端页面联调.
   新建ftl文件 类似templates/temptfl/commodity_add.ftl 
   
   templates/commodity_list.ftl
   templates/commodity_editor.ftl
   templates/commodity_add.ftl
   后台前台数据对接见注释
   
   Ajax 提交数据到后台
`    // 提交事件
               submitForm: function () {
                   var that = this;
                   var data = $('#myform').serializeObject();
                   var attr = {};
                   $('#attribute').find('tr').each(function(i) {
                       attr[$(this).find('.ipt').eq(0).val()] = $(this).find('.ipt').eq(1).val();
                   })
                   data.attribute = JSON.stringify(attr);
                   data.mark = 0;
                   $.ajax({
                       type: 'POST',
                       url: '/commodity/save',
                       data: JSON.stringify(data),
                       contentType: 'application/json',
                       beforeSend: function() {
                           that.enable = false;
                       },
                       success: function(res) {
                           res.status == 200 && $.notify({
                               type: 'success',
                               title: '添加成功',
                               callback: function() {
                                   location.href = '/commodity/list';
                               }
                           });
                       },
                       complete: function() {
                           that.enable = true;
                       }
                   })
               },`
 7. 自测一下提交服务器.在测试环境更新代码.发布一个测试环境的版本叫产品经理来验证.验证没有问题就发布了.  
   
   
##项目所用框架
1. spring boot
2. spring mvc
3. shiro
4. 模板引擎 freeMarker
5. mybatis

##项目相关注意点
1. 短信相关服务SmsUtil.(发送短信都是用这个工具类)
2. Dao层新建接口 例如PayRecordDao 需要添加 @AutoMapper 才能被加载
3. shiro 权限相关
    BossAuthorizationFilter -> isAccessAllowed 所有shiro 拦截的url都会调用这个方法.需要改变拦截逻辑时可以重写这个方法
    BossRealm -> doGetAuthenticationInfo 这个是用来获取验证的权限信息
    RetryLimitHashedCredentialsMatcher -> match 这个是shiro用来对比前台传进来的用户认证信息和后台数据库取出来的认证信息
    认证一致则表示登入成功
4. log 模块
   log项目地址 启动前需下载下来把log 包安装到本地
   https://github.com/zhouhang/compentent.git
   com.ms.boss.config.GetUser -> getLogUser 用来获取当前用户登入信息
   只有在controller 层加了 @BizLog(type = LogTypeConstant.COMMODITY, desc = "删除商品") 这个注解才会记录日志
5. 微信移动端相关的代码 控制层一般以Wechat,H5 开头例如WechatController 等
6. 防止重复提交  @SecurityToken(generateToken = true) 如下进入详情页时    @SecurityToken(generateToken = true)
保存商品信息时  @SecurityToken(validateToken = true)

`    /**
     * 商品详情
     * @param id
     * @param model
     * @return
     */
    @RequestMapping(value = "detail/{id}", method = RequestMethod.GET)
    @SecurityToken(generateToken = true)
    public String detail(@PathVariable("id") Integer id, ModelMap model) {
        // 获取单位信息
        CommodityVo vo = commodityService.findById(id);
        model.put("commodity", vo);
        return "commodity_editor";
    }
    /**
     * 保存
     * @param commodity
     * @return
     */
    @RequestMapping(value = "save", method = RequestMethod.POST)
    @ResponseBody
    @SecurityToken(validateToken = true)
    @BizLog(type = LogTypeConstant.COMMODITY, desc = "保存商品信息")
    public Result save(@RequestBody CommodityVo commodity) {
        Member mem= (Member) httpSession.getAttribute(RedisEnum.MEMBER_SESSION_BOSS.getValue());
        commodityService.save(commodity,mem.getId());
        return Result.success("保存成功!");
    }`
7. 微信提醒消息和短信提醒一般是通过 com.ms.service.observer.MsgProducerListener 之类的监听器来发送的
8. 本项目所有图片保存的都是绝对路径 保存和取出时都要经过转换. 示例代码如下
com.ms.service.impl.CommodityServiceImpl
从数据库读取商品信息把商品的图片读取出来 c.setThumbnailUrl(pathConvert.getUrl(c.getThumbnailUrl())); 
pathConvert.getUrl(c.getThumbnailUrl()) 把商品的绝对路径转换成相对路径
`       /**
        * 商品图片保存路径
        */
       private String folderName = "commodity/";
       @Override
       public PageInfo<CommodityVo> findByParams(CommodityVo commodityVo, Integer pageNum, Integer pageSize) {
           if (pageNum == null || pageSize == null) {
               pageNum = 1;
               pageSize = 10;
           }
           PageHelper.startPage(pageNum, pageSize);
           List<CommodityVo> list = commodityDao.findByParams(commodityVo);
           list.forEach(c->{
               c.setThumbnailUrl(pathConvert.getUrl(c.getThumbnailUrl()));
           });
           PageInfo page = new PageInfo(list);
           return page;
       }`
     保存商品图片 pathConvert.saveFileFromTemp(commodity.getPictureUrl(),folderName) 把上传的商品图片从临时文件保存到指定的目录
     `@Override
     @Transactional
     public void save(CommodityVo commodity,Integer memId) {
         Member mem= (Member) httpSession.getAttribute(RedisEnum.MEMBER_SESSION_BOSS.getValue());
         commodity.setPictureUrl(pathConvert.saveFileFromTemp(commodity.getPictureUrl(),folderName));`
         
9. 图片上传 参考GeneralController 里面调用的UploadServiceImpl 里面包含的加水印等功能

`    /**
     * 上传商品图片信息
     * @param img
     * @return
     */
    @RequestMapping(value = "/upload", method = RequestMethod.POST)
    @ResponseBody
    public CropResult updateFile(@RequestParam(required = false) MultipartFile img) throws Exception{
        return uploadService.uploadImage(img);
    }`
    
10. excel 解析和导入com.ms.service.utils.ExcelParse 里面有excel 相关的导入和解析代码

11. 商品JS代码模块规范

