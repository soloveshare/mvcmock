### SpringMVC-rest-test
[![Build Status](https://www.travis-ci.org/soloveshare/mvcmock.svg?branch=master)](https://www.travis-ci.org/soloveshare/mvcmock)

[![Coverage Status](https://coveralls.io/repos/github/soloveshare/mvcmock/badge.svg?branch=master)](https://coveralls.io/github/soloveshare/mvcmock?branch=master)

This is a very basic example of using Spring MVC, REST and Spring Test Framework using Spring's Java configuration.

The first part of this is to create a configuration class for the web app.  below is a sample of the configuration class we are going to use:


    @Configuration
    @EnableWebMvc
    @ComponentScan(basePackages = {"com.johnathanmsmith.mvc.web"})
    public class WebMVCConfiguration extends WebMvcConfigurerAdapter
    {

        private static final Logger logger = LoggerFactory.getLogger(WebMVCConfiguration.class);

        @Bean
        public ViewResolver resolver()
        {
            UrlBasedViewResolver url = new UrlBasedViewResolver();
            url.setPrefix("/views/");
            url.setViewClass(JstlView.class);
            url.setSuffix(".jsp");
            return url;
        }

        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry)
        {
            logger.debug("setting up resource handlers");
            registry.addResourceHandler("/resources/").addResourceLocations("/resources/**");
        }

        @Override
        public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer)
        {
            logger.debug("configureDefaultServletHandling");
            configurer.enable();
        }

        @Bean
        public SimpleMappingExceptionResolver simpleMappingExceptionResolver()
        {
            SimpleMappingExceptionResolver b = new SimpleMappingExceptionResolver();

            Properties mappings = new Properties();
            mappings.put("org.springframework.web.servlet.PageNotFound", "p404");
            mappings.put("org.springframework.dao.DataAccessException", "dataAccessFailure");
            mappings.put("org.springframework.transaction.TransactionException", "dataAccessFailure");
            b.setExceptionMappings(mappings);
            return b;
        }
    }


Next you have to setup the web.xml file to use the above configuration class, we do this but setting the contectConfigLocation to the package of the configuration class. see below:

    <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns="http://java.sun.com/xml/ns/javaee"
             xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
             version="2.5">

        <servlet>
            <servlet-name>Spring MVC Dispatcher Servlet</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
            <init-param>
                <param-name>contextClass</param-name>
                <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
            </init-param>
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>com.johnathanmsmith.mvc.web.config, com.johnathanmsmith.mvc.web.controller</param-value>
            </init-param>
            <load-on-startup>1</load-on-startup>
        </servlet>

        <servlet-mapping>
            <servlet-name>Spring MVC Dispatcher Servlet</servlet-name>
            <url-pattern>/</url-pattern>
        </servlet-mapping>
    </web-app>

Now lets setup a basic controller to display a page:

    @Controller
    @RequestMapping("/json")
    class JSonController
    {

        private static final Logger logger = LoggerFactory.getLogger(JSonController.class);


        @RequestMapping(value = "/{name}", method = RequestMethod.GET)
        @ResponseBody
        public User getName(@PathVariable String name, ModelMap model) throws ResourceNotFoundException
        {

            logger.debug("I am in the controller and got user name: " + name);

            /*

                Simulate a successful lookup for 2 users, this is where your real lookup code would go

             */

            if ("JohnathanMarkSmith".equals(name))
            {
                return new User("Johnathan Mark Smith", name);
            }

            if ("Regan".equals(name))
            {
                return new User("Regan Smith", name);
            }

            throw new ResourceNotFoundException("User Is Not Found");
        }

        @RequestMapping(value = "/", method = RequestMethod.GET)
        @ResponseBody
        public ResponseEntity<User> getDisplayDefault(ModelMap model)
        {
                /*
                    you did not enter a name so the default is going to run
                 */
                return new ResponseEntity<User>(new User("Johnathan Mark Smith", "JohnathanMarkSmith"), HttpStatus.OK);
        }

        @ExceptionHandler
        @ResponseBody
        public ResponseEntity<ErrorHolder> handle(ResourceNotFoundException e)
        {
            logger.warn("The resource was not found", e);
            return new ResponseEntity<ErrorHolder>(new ErrorHolder("The resource was not found"), HttpStatus.NOT_FOUND);
        }

    }

## Testing Your Web Service

Below you will see the Spring Test Framework and how to tell your web server with it

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(classes = {WebMVCConfiguration.class})
    @WebAppConfiguration
    public class TestHelloWorldWeb
    {
        @Autowired
        private WebApplicationContext wac;

        private MockMvc mockMvc;

        @Before
        public void setup()
        {
            this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
        }

        @Test
        public void getHTML() throws Exception
        {
            /*
                This following code will do 'GET' to the web apps
                and check that the return view is "helloworld"
                and also that it has a attribute "user" to "JohnathanMarkSmith"

             */
            this.mockMvc.perform(get("/ask/JohnathanMarkSmith")
                    .accept(MediaType.TEXT_HTML))
                    .andExpect(status().isOk())
                    .andExpect(view().name("helloworld"))
                    .andExpect(MockMvcResultMatchers.model().attribute("user", "JohnathanMarkSmith"));


        }

        @Test
        public void getJSON() throws Exception
        {
            /*
                This following code will do 'GET' to the web apps
                and also that it has a attribute "user" to "JohnathanMarkSmith"

             */
            this.mockMvc.perform(get("/json/JohnathanMarkSmith")
                    .accept(MediaType.APPLICATION_JSON))
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.user").value("Johnathan Mark Smith"));
        }
    }


Thats all it takes..

## Getting The Project and Running It

To get this project and run it you will need to follow the following steps:

    git clone  git@github.com:JohnathanMarkSmith/springmvc-rest-test.git
    cd springmvc-rest-test/
    mvn tomcat7:run

Now open your web brower and goto http://127.0.0.1:8080/springmvc-rest-test/

This its... Have run with it...


If you have any questions or comments please email me at john@johnathanmarksmith.com or checkout my web site http://JohnathanMarkSmith.com

