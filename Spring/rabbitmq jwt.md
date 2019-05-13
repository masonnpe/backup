    @Controller
    @RequestMapping("/good")
    public class GoodController {
        @Autowired
        TemplateEngine templateEngine;
    
        @Resource
        private AmqpTemplate amqpTemplate;
    
        @RequestMapping("/message/{type}/{msg}")
        @ResponseBody
        public String testSend(@PathVariable String type,@PathVariable String msg) throws InterruptedException {
          //  this.amqpTemplate.convertAndSend("spring.test.exchange","a.b", msg);
            this.amqpTemplate.convertAndSend("good.exchange","good."+type, msg);
            return "";
        }
    
    
    
        @RequestMapping("/{id}")
        public String toHtml(@PathVariable Long id, Model model){
            System.out.println("来过");
            Map<String,Object> map=new HashMap<>();
            map.put("message","hello");
            updateToNginxPath(map,"good","good"+id);
            model.addAllAttributes(map);
            return "good";
        }
    
        public void updateToNginxPath(Map map,String templateName,String localName){
            PrintWriter writer = null;
            try {
                Context context = new Context();
                context.setVariables(map);
                File file = new File("E:\\nginx-1.15.12\\nginx-1.15.12\\html\\" + localName + ".html");
                writer = new PrintWriter(file);
                templateEngine.process(templateName, context, writer);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (writer != null) {
                    writer.close();
                }
            }
        }
    
    
    }





    @Component
    public class Consumer {
        @RabbitListener(bindings = @QueueBinding(
                value = @Queue(value = "add.queue", durable = "true"),
                exchange = @Exchange(
                        value = "good.exchange",
                        ignoreDeclarationExceptions = "true",
                        type = ExchangeTypes.TOPIC
                ), key = "good.add"))
        public void add(String msg){
            System.out.println("add：" + msg);
        }
    
        @RabbitListener(bindings = @QueueBinding(
                value = @Queue(value = "delete.queue", durable = "true"),
                exchange = @Exchange(
                        value = "good.exchange",
                        ignoreDeclarationExceptions = "true",
                        type = ExchangeTypes.TOPIC
                ), key = "good.delete"))
        public void delete(String msg){
            System.out.println("delete：" + msg);
        }
    
    }



    spring.thymeleaf.mode=HTML5
    spring.thymeleaf.cache=false
    spring.rabbitmq.host=192.168.170.128
    spring.rabbitmq.username=hehe
    spring.rabbitmq.password=hehe
    spring.rabbitmq.virtual-host=/steambuy
    spring.rabbitmq.publisher-confirms=true

@Cookievalue



    public class Test {
    
        public static void main(String[] args) {
            JwtBuilder claim = Jwts.builder().setId("12312").setSubject("dadaddadsad")
                    .setIssuedAt(new Date()).signWith(SignatureAlgorithm.HS256, "steambuy").setExpiration(new Date(new Date().getTime() + 60000)).claim("role", "");
            System.out.println(claim.compact());
    
    
            Claims steambuy = (Claims)Jwts.parser().setSigningKey("steambuy").parse(claim.compact()).getBody();
            System.out.println(steambuy.getId()+steambuy.getSubject());
        }
    }





    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
        <version>0.9.0</version>
    </dependency>
    <dependency>
        <groupId>joda-time</groupId>
        <artifactId>joda-time</artifactId>
    </dependency>


