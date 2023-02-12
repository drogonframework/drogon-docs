[English](ENG-05-Filter) | [简体中文](CHN-05-过滤器)

In HttpController's [example](ENG-04-2-Controller-HttpController), the getInfo method should check whether the user is logged in before returning the user's information. We can write this logic in the getInfo method, but obviously, checking the user's login membership is general logic which will be used by many interfaces, it should be extracted separately and configured before calling handler, which is what filters do.
After the drogon framework completes the URL path matching, it first calls the filters registered on the path in turn, and only when all the filters allow "pass", the corresponding handler will be called;

### Built-in Filter

Drogon contains the following common filters:

- `drogon::IntranetIpFilter`: allow HTTP requests from intranet IP only, or return the 404 page.
- `drogon::LocalHostFilter`: allow HTTP requests from 127.0.0.1 or ::1 only, or return the 404 page.

### Custom Filter

- #### Filter Definition

  Of course, users can customize the filter, you need to inherit the HttpFilter class template, the template type is the subclass type, for example, if you want to create a LoginFilter, you could define it as follows:

  ```c++
  class LoginFilter:public drogon::HttpFilter<LoginFilter>
  {
  public:
      virtual void doFilter(const HttpRequestPtr &req,
                          FilterCallback &&fcb,
                          FilterChainCallback &&fccb) override ;
  };
  ```

  You could create filter by the `drogon_ctl` command, see [drogon_ctl](ENG-11-drogon_ctl-command#Filter-creation).

  You need to override the doFilter virtual function of the parent class to implement the filter logic;

  This virtual function has three parameters, which are:

  - **req**: http request;
  - **fcb**: filter callback function, the function type is void (HttpResponsePtr), when the filter determines that the request is not valid, the specific response is returned to the browser through this callback;
  - **fccb**: filter chain callback function, the function type is void (), when the filter determines that the request is legal, tells drogon to call the next filter or the final handler through this callback;

  The specific implementation can refer to the implementation of the drogon built-in filter.

- #### Filter Registration

  The registration of filters is always accompanied by the registration of controllers.the macros (PATH_ADD, METHOD_ADD, etc.) mentioned earlier can add the name of one or more filters at the end; for example, we change the registration line of the previous `getInfo` method to the following form:

  ```c++
  METHOD_ADD(User::getInfo,"/{1}/info?token={2}",Get,"LoginFilter");
  ```

  After the path is successfully matched, the `getInfo` method will be called only when the following conditions were met:

  1. The request must be an HTTP Get request;
  2. The requesting party must have logged in;

  As you can see, the configuration and registration of filters are very simple. The controller source file that registers filters does not need to include the filter's header file. The filter and controller are fully decoupled.

  > **Note: If the filter is defined in the namespace, you must write the namespace completely when you register the filter.**

# 06 [View](ENG-06-View)
