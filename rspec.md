# rspec5 文档整理

## 安装

##### 添加到 Gemfile

```ruby
$ echo 'gem "rspec-rails", group: [:development, :test]' >> Gemfile
```

##### 安装

```ruby
$ bundle install
```

##### 初始化成为默认测试框架

```ruby
$ rails generate rspec:install
```

##### 运行测试

```ruby
$ rails spec
```

##### 或者

```ruby
$ rspec spec --format documentation
```



## 测试框架

```ruby
# spec/failing_spec.rb
require "rails_helper"

# 配置
RSpec.configure do |config|
  config.xxx
  ...
end

# 类级别命名空间
RSpec.describe "类名", :type => :类型 do
  # 使用内部的匿名控制器
  controller do
    def 方法名
      # 实现
    end
  end
    
  before(:context) do
    @widget = Widget.create!
  end

  after(:context) do
    @widget.destroy
  end

  # 类的方法级别命名空间
  # describe "请求方法 方法名" 
  describe "GET index" do
    before(:context) do
      @widget = Widget.create!
    end

    after(:context) do
      @widget.destroy
    end
    
    # 具体的测试用例 允许多个，包含成功的和失败的
    it "raises an error" do
      # 具体实现
      get :index
    end
  end
end
```



## 内置创建器

```ruby
$ rails generate rspec:model user
$ rails generate rspec:controller api::v1::users
```

> 生成文件 spec/models/user_spec.rb

更多的 specs 标签支持:

- `scaffold`
- `model`
- `controller`
  - `rails g rspec:controller api::v1::users`
- `helper`
- `view`
- `mailer`
- `integration`
- `feature`
- `job`
- `channel`
- `generator`
- `mailbox`
- `request`
- `system`



## 事务

rspec默认开启事务，这样可以保证测试完成后自动清空数据库。当然可以使用其他gem来完成这个功能，例如 [database_cleaner](https://github.com/bmabey/database_cleaner) 。

当运行 `rails generate rspec:install`, 在文件 `spec/rails_helper.rb` 中默认引入了系列配置：

```ruby
RSpec.configure do |config|
  config.use_transactional_fixtures = true
end
```

##### 禁用事务

```ruby
RSpec.configure do |config|
  config.use_transactional_fixtures = false
end
```



##### 利用 `before` 关键字实现自动回滚的控制

- 自动回滚：任何在 `before(:example)` 定义的数据在执行完测试后都会自动清空。

  ```ruby
  describe Widget do
    before(:example) do
      @widget = Widget.create
    end
  
    it "does something" do
      expect(@widget).to do_something
    end
  
    it "does something else" do
      expect(@widget).to do_something_else
    end
  end
  ```

   `@widget` 在每个 it 中都会被重新创建，所以互不干扰。



- 手动控制：在 `before(:context)` 中创建的数据则不会自动回滚。

  你可以利用`before(:context)` 创建多个方法的公用数据。下面是几个原则：

  1. 一定要定义 `after(:context)` 来手动删除数据:

     ```ruby
     before(:context) do
       @widget = Widget.create!
     end
     
     after(:context) do
       @widget.destroy
     end
     ```

     否则你的数据会一直存在有可能影响其他测试。

  2. 一定要使用 `before(:example)` 刷新数据。

     ```ruby
     before(:context) do
       @widget = Widget.create!
     end
     
     before(:example) do
       @widget.reload
     end
     ```

     



## 目录结构建议

##### rspec的目录规范

| 模块               | 所在目录                                      |
| ------------------ | --------------------------------------------- |
| `Model specs`      | `spec/models`                                 |
| `Controller specs` | `spec/controllers`                            |
| `Request specs`    | `spec/requests`  或者 `integration` 或者`api` |
| `Feature specs`    | `spec/features`                               |
| `View specs`       | `spec/views`                                  |
| `Helper specs`     | `spec/helpers`                                |
| `Mailer specs`     | `spec/mailers`                                |
| `Routing specs`    | `spec/routing`                                |
| `Job specs`        | `spec/jobs`                                   |
| `System specs`     | `spec/system`                                 |

当然开发人员可以任意自定义目录结构，这个时候就需要明确的说明当前的测试是针对那个模块儿。如何指定呢？使用`type`, 在rspec中可用的type类型有：

| 模块               | 使用类型标示        |
| ------------------ | ------------------- |
| `Model specs`      | `type: :model`      |
| `Controller specs` | `type: :controller` |
| `Request specs`    | `type: :request`    |
| `Feature specs`    | `type: :feature`    |
| `View specs`       | `type: :view`       |
| `Helper specs`     | `type: :helper`     |
| `Mailer specs`     | `type: :mailer`     |
| `Routing specs`    | `type: :routing`    |
| `Job specs`        | `type: :job`        |
| `System specs`     | `type: :system`     |

举个例子：

```ruby
# spec/legacy/things_controller_spec.rb
# 明确表明了要测试的类型
RSpec.describe ThingsController, type: :controller do
  describe "GET index" do
    ​# Examples
  end
end
```



##### 自动识别类型

如果目录的安排是按照rspec规范的目录，可以通过配置自动的识别类型

```ruby
# spec/rails_helper.rb
RSpec.configure do |config|
  config.infer_spec_type_from_file_location!
end
```

但是如果是自定义目录，也可以这样做：

```ruby
# set `:type` for serializers directory
RSpec.configure do |config|
  config.define_derived_metadata(:file_path => Regexp.new('/spec/serializers/')) do |metadata|
    metadata[:type] = :serializer
  end
end
```



##### 建议

强烈推荐按照规范罗列目录，例如：

```ruby
app
├── controllers
│   ├── application_controller.rb
│   └── books_controller.rb
├── helpers
│   ├── application_helper.rb
│   └── books_helper.rb
├── models
│   ├── author.rb
│   └── book.rb
└── views
    ├── books
    └── layouts
lib
├── country_map.rb
├── development_mail_interceptor.rb
├── enviroment_mail_interceptor.rb
└── tasks
    └── irc.rake
spec
├── controllers
│   └── books_controller_spec.rb
├── country_map_spec.rb
├── features
│   └── tracking_book_delivery_spec.rb
├── helpers
│   └── books_helper_spec.rb
├── models
│   ├── author_spec.rb
│   └── book_spec.rb
├── rails_helper.rb
├── requests
│   └── books_spec.rb
├── routing
│   └── books_routing_spec.rb
├── spec_helper.rb
├── tasks
│   └── irc_spec.rb
└── views
    └── books
```

##### 场景

- 自动识别测试类型，添加 type

  ```ruby
  #spec/functional/widgets_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe WidgetsController, :type => :controller do
    it "responds successfully" do
      get :index
      expect(response.status).to eq(200)
    end
  end
  ```

- 非Rails应用就不要添加 type

  ```ruby
  #spec/ledger/entry_spec.rb
  require "spec_helper"
  
  Entry = Struct.new(:description, :us_cents)
  
  RSpec.describe Entry do
    it "has a description" do
      is_expected.to respond_to(:description)
    end
  end
  ```

- 从文件位置推断规范类型会添加适当的元数据

  ```ruby
  #spec/controllers/widgets_controller_spec.rb
  require "rails_helper"
  
  RSpec.configure do |config|
    config.infer_spec_type_from_file_location!
  end
  
  RSpec.describe WidgetsController do
    it "responds successfully" do
      get :index
      expect(response.status).to eq(200)
    end
  end
  ```

- 规范目录中的规范可以覆盖其推断的类型

  ```ruby
  # spec/routing/duckduck_routing_spec.rb
  require "rails_helper"
  
  Rails.application.routes.draw do
    get "/example" => redirect("http://example.com")
  end
  
  RSpec.configure do |config|
    config.infer_spec_type_from_file_location!
  end
  
  # Due to limitations in the Rails routing test framework, routes that
  # perform redirects must actually be tested via request specs
  RSpec.describe "/example", :type => :request do
    it "redirects to example.com" do
      get "/example"
      expect(response).to redirect_to("http://example.com")
    end
  end
  ```



## Backtrace 过滤

Rails中有些gem会让失败输出的 backtrace 信息会被过滤和简化，我们可以通过配置让失败的输出全部的 backtrace：

```ruby
RSpec.configure do |config|
  config.filter_rails_from_backtrace!
end
```



##### 局部使用 `filter_rails_from_backtrace!`

```ruby
# spec/failing_spec.rb
require "rails_helper"

RSpec.configure do |config|
  config.filter_rails_from_backtrace!
end

RSpec.describe "Controller", :type => :controller do
  # 使用内部的匿名控制器
  controller do
    def index
      raise "Something went wrong."
    end
  end

  describe "GET index" do
    it "raises an error" do
      get :index
    end
  end
end
```



##### 场景

使用 `rspec` 

```ruby
- When I run `rspec`

- Then
  the output should contain "1 example, 1 failure"

- And
  the output should not contain "activesupport"
```



使用 `rspec --backtrace`

```ruby
- I run `rspec --backtrace`

- Then
  the output should contain "1 example, 1 failure"

- And
  the output should contain "activesupport"
```





## 为模型编写rspec

模型的rspec是ActiveSupport :: TestCase的美化包装，并且包括所有它提供的行为和`断言`，以及RSpec自身的行为和`期望`。

例子:

```ruby
require "rails_helper"
# 模型是：Post， 明确指定了type
RSpec.describe Post, :type => :model do
  # 命名空间是 with 2 or more comments
  context "with 2 or more comments" do
    # 真正的测试
    it "orders them in reverse chronologically" do
      # 调用create方法创建了一个 post
      post = Post.create!
      # 创建一个回复
      comment1 = post.comments.create!(:body => "first comment")
      # 创建第二个回复
      comment2 = post.comments.create!(:body => "second comment")
      # 期望所有回复中 comment2, comment1相等
      expect(post.reload.comments).to eq([comment2, comment1])
    end
  end
end
```



##### 事务的例子

- 默认使用事务

  ```ruby
  # spec/models/widget_spec.rb
  require "rails_helper"
  
  # 验证模型 Widget
  RSpec.describe Widget, :type => :model do
    # 没有数据
    it "has none to begin with" do
      expect(Widget.count).to eq 0
    end
  
    # 添加一个
    it "has one after adding one" do
      Widget.create
      expect(Widget.count).to eq 1
    end
  
    # 因为有事务 所以这个数据是独立的  也成立
    it "has none after one was created in a previous example" do
      expect(Widget.count).to eq 0
    end
  end
  ```

  运行测试：

  ```shell
  $ rspec spec/models/widget_spec.rb
  ```

  应该都通过。

- 明确指定事务开启

  ```ruby
  #spec/models/widget_spec.rb
  require "rails_helper"
  
  RSpec.configure do |c|
    c.use_transactional_examples = true
  end
  
  RSpec.describe Widget, :type => :model do
    it "has none to begin with" do
      expect(Widget.count).to eq 0
    end
  
    it "has one after adding one" do
      Widget.create
      expect(Widget.count).to eq 1
    end
  
    it "has none after one was created in a previous example" do
      expect(Widget.count).to eq 0
    end
  end
  ```

  运行测试：

  ```shell
  $ rspec spec/models/widget_spec.rb
  ```

  应该都通过。

- 明确禁用事务

  ```ruby
  #spec/models/widget_spec.rb" with:
  require "rails_helper"
  
  RSpec.configure do |c|
    # 关闭事务 
    c.use_transactional_examples = false
    # 设定公用值
    c.order = "defined"
  end
  
  RSpec.describe Widget, :type => :model do
    it "has none to begin with" do
      expect(Widget.count).to eq 0
    end
  
    it "has one after adding one" do
      Widget.create
      expect(Widget.count).to eq 1
    end
  
    it "has one after one was created in a previous example" do
      expect(Widget.count).to eq 1
    end
  
    # 利用after删除
    after(:all) { Widget.destroy_all }
  end
  ```

  运行测试：

  ```shell
  $ rspec spec/models/widget_spec.rb
  ```

  应该都通过。

- 使用 fixture 预定值

  ```ruby
  #spec/models/thing_spec.rb
  require "rails_helper"
  
  RSpec.describe Thing, :type => :model do
    fixtures :things
    it "fixture method defined" do
      things(:one)
    end
  end
  
  
  #spec/fixtures/things.yml
  one:
    name: MyString
  ```

  运行测试：

  ```shell
  $ rspec spec/models/thing_spec.rb
  ```

  应该都通过。

##### 支持 double

默认情况下，rspec验证的double不支持动态方法。而instance_double.rspec-rails通过扩展的方法为列启用了此支持。

例子：

```ruby
# spec/models/widget_spec.rb
require "rails_helper"

RSpec.describe Widget, :type => :model do
  it "has one after adding one" do
    # 假造一个对象Widget 属性为name
    instance_double("Widget", :name => "my name")
  end
end
```



























## 为控制器编写rspec

controller spec 继承自  `ActionController::TestCase::Behavior`, 允许你发送http请求并获取相应的期望:

- rendered templates
- redirects
- instance variables assigned in the controller to be shared with the view
- cookies sent back with the response

可以像这样制定结果:

- 标准匹配： (`expect(response.status).to eq(200)`)

- 标准断言： (`assert_equal 200, response.status`)

- rails 断言 (`assert_response 200`)

- rails-specific 匹配器:

  - 期望渲染了模板

    ```
    expect(response).to render_template(:new)   # wraps assert_template
    ```

  - 期望执行了重定向

    ```
    expect(response).to redirect_to(location)   # wraps assert_redirected_to
    ```

  - 期望得到指定的状态值

    ```
    expect(response).to have_http_status(:created)
    ```

  - 期待新创建了对象

    ```
    expect(assigns(:widget)).to be_a_new(Widget)
    ```

简单实例：

```ruby
RSpec.describe TeamsController do
  describe "GET index" do
    it "assigns @teams" do
      team = Team.create
      get :index
      expect(assigns(:teams)).to eq([team])
    end

    it "renders the index template" do
      get :index
      expect(response).to render_template("index")
    end
  end
end
```

指定头信息

```ruby
require "rails_helper"

RSpec.describe TeamsController, type: :controller do
  describe "GET index" do
    it "returns a 200" do
      # 添加头信息
      request.headers["Authorization"] = "foo"
      # 请求show方法
      get :show
      # 期待返回值是 200
      expect(response).to have_http_status(:ok)
    end
  end
end
```



### controller 相关

- 简单实例

  ```ruby
  # spec/controllers/widgets_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe WidgetsController, :type => :controller do
    describe "GET index" do
      it "has a 200 status code" do
        get :index
        # 期待状态码的值是 200
        expect(response.status).to eq(200)
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 测试返回值的数据类型

  ```ruby
  # spec/controllers/widgets_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe WidgetsController, :type => :controller do
    describe "responds to" do
      it "responds to html by default" do
        post :create, :params => { :widget => { :name => "Any Name" } }
        # 返回值为类型为 "text/html"
        expect(response.content_type).to eq "text/html"
      end
  
      it "responds to custom formats when provided in the params" do
        post :create, :params => { :widget => { :name => "Any Name" }, :format => :json }
        # 返回值为类型为 "text/html"
        expect(response.content_type).to eq "application/json"
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 指定请求参数的数据类型

  ```ruby
  # spec/controllers/widgets_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe WidgetsController, :type => :controller do
    describe "responds to" do
      it "responds to html by default" do
        post :create, :params => { :widget => { :name => "Any Name" } }
        expect(response.content_type).to eq "text/html"
      end
  
      it "responds to custom formats when provided in the params" do
        # 设定请求参数类型为 json
        post :create, :params => { :widget => { :name => "Any Name" }, :format => :json }
        expect(response.content_type).to eq "application/json"
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 测试返回值的数据编码类型

  ```ruby
  #spec/controllers/widgets_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe WidgetsController, :type => :controller do
    describe "responds to" do
      it "responds to html by default" do
        post :create, :params => { :widget => { :name => "Any Name" } }
        # 编码类型为 text/html; charset=utf-8
        expect(response.content_type).to eq "text/html; charset=utf-8"
      end
  
      it "responds to custom formats when provided in the params" do
        post :create, :params => { :widget => { :name => "Any Name" }, :format => :json }
        # 编码类型为 application/json; charset=utf-8
        expect(response.content_type).to eq "application/json; charset=utf-8"
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 测试返回值的 media_type

  ```ruby
  # spec/controllers/widgets_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe WidgetsController, :type => :controller do
    describe "responds to" do
      it "responds to html by default" do
        post :create, :params => { :widget => { :name => "Any Name" } }
        #media_type 为 text/html
        expect(response.media_type).to eq "text/html"
      end
  
      it "responds to custom formats when provided in the params" do
        post :create, :params => { :widget => { :name => "Any Name" }, :format => :json }
        expect(response.media_type).to eq "application/json"
      end
    end
  end
  ```
  
  运行测试
  
  ```shell
  $ rspec spec
  ```
  
  all通过测试



### 指定具体的视图文件

- 期望通过控制器动作呈现的模板（通过）

  ```ruby
  # spec/controllers/widgets_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe WidgetsController, :type => :controller do
    describe "index" do
      it "renders the index template" do
        get :index
        # 测试渲染了指定的 index 模板
        expect(response).to render_template("index")
        # 测试返回值为 “”
        expect(response.body).to eq ""
      end
      it "renders the widgets/index template" do
        get :index
        expect(response).to render_template("widgets/index")
        expect(response.body).to eq ""
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 期望控制器操作未呈现的模板（失败）

  ```ruby
  # spec/controllers/widgets_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe WidgetsController, :type => :controller do
    describe "index" do
      it "renders the 'new' template" do
        get :index
        expect(response).to render_template("new")
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  没有通过测试

- 期望在运行时更改视图路径时呈现空模板（通过）

  ```ruby
  # spec/controllers/things_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe ThingsController, :type => :controller do
    describe "custom_action" do
      it "renders an empty custom_action template" do
        # 更改视图路径
        controller.prepend_view_path 'app/views'
        controller.append_view_path 'app/views'
        get :custom_action
        expect(response).to render_template("custom_action")
        expect(response.body).to eq ""
      end
    end
  end
  
  
  #app/controllers/things_controller.rb
  class ThingsController < ActionController::Base
    layout false
    def custom_action
    end
  end
  
  #app/views/things/custom_action.html.erb
  # 空文件
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 在运行时更改视图路径时，期望模板使用render_views渲染实际模板（通过）

  ```ruby
  # spec/controllers/things_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe ThingsController, :type => :controller do
    render_views
  
    it "renders the real custom_action template" do
      controller.prepend_view_path 'app/views'
      get :custom_action
      expect(response).to render_template("custom_action")
      expect(response.body).to match(/template for a custom action/)
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试



### 匹配渲染绘图内容

- 直接在单个组中进行render_views

  ```ruby
  # spec/controllers/widgets_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe WidgetsController, :type => :controller do
    render_views
  
    describe "GET index" do
      it "has a widgets related heading" do
        get :index
        expect(response.body).to match /<h1>.*widgets/im
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 在嵌套组中打开和关闭render_views

  ```ruby
  # spec/controllers/widgets_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe WidgetsController, :type => :controller do
    context "with render_views" do
      render_views
  
      describe "GET index" do
        it "renders the actual template" do
          get :index
          expect(response.body).to match /<h1>.*widgets/im
        end
      end
  
      context "with render_views(false) nested in a group with render_views" do
        render_views false
  
        describe "GET index" do
          it "renders the RSpec generated template" do
            get :index
            expect(response.body).to eq("")
          end
        end
      end
    end
  
    context "without render_views" do
      describe "GET index" do
        it "renders the RSpec generated template" do
          get :index
          expect(response.body).to eq("")
        end
      end
    end
  
    context "with render_views again" do
      render_views
  
      describe "GET index" do
        it "renders the actual template" do
          get :index
          expect(response.body).to match /<h1>.*widgets/im
        end
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec --order default --format documentation
  WidgetsController
    with render_views
      GET index
        renders the actual template
      with render_views(false) nested in a group with render_views
        GET index
          renders the RSpec generated template
    without render_views
      GET index
        renders the RSpec generated template
    with render_views again
      GET index
        renders the actual template
  ```

  all通过测试

- 全局render_views

  ```ruby
  # spec/support/render_views.rb" with:
  RSpec.configure do |config|
    config.render_views
  end
  
  
  # spec/controllers/widgets_controller_spec.rb
  require "rails_helper"
  require "support/render_views"
  
  RSpec.describe WidgetsController, :type => :controller do
    describe "GET index" do
      it "renders the index template" do
        get :index
        expect(response.body).to match /<h1>.*widgets/im
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试



### 匿名控制器

使用controller方法定义一个匿名控制器，该匿名控制器将从传入的参数所描述的类继承。这对于指定类似全局错误处理的行为很有用。

要指定其他基类，您可以将该类显式传递给 controller 方法：

```ruby
controller(BaseController)
```

您还可以禁用基本类型推断，在这种情况下，匿名控制器将默认继承自ApplicationController方法而不是所描述的类。

```ruby
RSpec.configure do |c|
  # 禁用基本类型推断
  c.infer_base_class_for_anonymous_controllers = false
end

RSpec.describe BaseController, :type => :controller do
  	# ApplicationController
    controller do
    def index; end

    # this normally creates an anonymous `BaseController` subclass,
    # however since `infer_base_class_for_anonymous_controllers` is
    # disabled, it creates a subclass of `ApplicationController`
  end
end
```



- 使用重定向处理在`ApplicationController`中指定的错误

  ```ruby
  # spec/controllers/application_controller_spec.rb
  require "rails_helper"
  
  class ApplicationController < ActionController::Base
    class AccessDenied < StandardError; end
  
    rescue_from AccessDenied, :with => :access_denied
  
  private
  
    def access_denied
      redirect_to "/401.html"
    end
  end
  
  RSpec.describe ApplicationController, :type => :controller do
    controller do
      def index
        raise ApplicationController::AccessDenied
      end
    end
  
    describe "handling AccessDenied exceptions" do
      it "redirects to the /401.html page" do
        get :index
        expect(response).to redirect_to("/401.html")
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 使用 render 处理在`ApplicationController`中指定的错误

  ```ruby
  # spec/controllers/application_controller_spec.rb
  require "rails_helper"
  
  class ApplicationController < ActionController::Base
    class AccessDenied < StandardError; end
  
    rescue_from AccessDenied, :with => :access_denied
  
  private
  
    def access_denied
      render "errors/401"
    end
  end
  
  RSpec.describe ApplicationController, :type => :controller do
    controller do
      def index
        raise ApplicationController::AccessDenied
      end
    end
  
    describe "handling AccessDenied exceptions" do
      it "renders the errors/401 template" do
        get :index
        expect(response).to render_template("errors/401")
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 使用 render_template 处理在`ApplicationController`中指定的错误

  ```ruby
  # spec/controllers/application_controller_spec.rb
  require "rails_helper"
  
  class ApplicationController < ActionController::Base
    class AccessDenied < StandardError; end
  
    rescue_from AccessDenied, :with => :access_denied
  
  private
  
    def access_denied
      render :file => "errors/401"
    end
  end
  
  RSpec.describe ApplicationController, :type => :controller do
    controller do
      def index
        raise ApplicationController::AccessDenied
      end
    end
  
    describe "handling AccessDenied exceptions" do
      it "renders the errors/401 template" do
        get :index
        expect(response).to render_template("errors/401")
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 使用 子类 处理在`ApplicationController`中指定的错误

  ```ruby
  # spec/controllers/application_controller_subclass_spec.rb
  require "rails_helper"
  
  class ApplicationController < ActionController::Base
    class AccessDenied < StandardError; end
  end
  
  class FoosController < ApplicationController
  
    rescue_from ApplicationController::AccessDenied, with: :access_denied
  
  private
  
    def access_denied
      redirect_to "/401.html"
    end
  end
  
  RSpec.describe FoosController, :type => :controller do
    controller(FoosController) do
      def index
        raise ApplicationController::AccessDenied
      end
    end
  
    describe "handling AccessDenied exceptions" do
      it "redirects to the /401.html page" do
        get :index
        expect(response).to redirect_to("/401.html")
      end
    end
  end
  ```
  
  运行测试
  
  ```shell
  $ rspec spec
  ```
  
  all通过测试
  
- 从描述的类中推断基类

  ```ruby
  # spec/controllers/base_class_can_be_inferred_spec.rb
  require "rails_helper"
  
  class ApplicationController < ActionController::Base; end
  
  class FoosController < ApplicationController; end
  
  RSpec.describe FoosController, :type => :controller do
    controller do
      def index
        render :plain => "Hello World"
      end
    end
  
    it "creates anonymous controller derived from FoosController" do
      expect(controller).to be_a_kind_of(FoosController)
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 在描述的类中使用`name`和`controller_name`

  ```ruby
  # spec/controllers/get_name_and_controller_name_from_described_class_spec.rb
  require "rails_helper"
  
  class ApplicationController < ActionController::Base; end
  class FoosController < ApplicationController; end
  
  RSpec.describe "Access controller names", :type => :controller do
    controller FoosController do
      def index
        @name = self.class.name
        @controller_name = controller_name
        render :plain => "Hello World"
      end
    end
  
    before do
      get :index
    end
  
    it "gets the class name as described" do
      expect(assigns[:name]).to eq('FoosController')
    end
  
    it "gets the controller_name as described" do
      expect(assigns[:controller_name]).to eq('foos')
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 在基类中调用`around_filter`和`around_action`

  ```ruby
  # spec/controllers/application_controller_around_filter_spec.rb
  require "rails_helper"
  
  class ApplicationController < ActionController::Base
    around_action :an_around_filter
  
    def an_around_filter
      @callback_invoked = true
      yield
    end
  end
  
  RSpec.describe ApplicationController, :type => :controller do
    controller do
      def index
        render :plain => ""
      end
    end
  
    it "invokes the callback" do
      get :index
  
      expect(assigns[:callback_invoked]).to be_truthy
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 匿名控制器仅创建资源路由

  ```ruby
  # spec/controllers/application_controller_spec.rb
  require "rails_helper"
  
  if defined?(ActionController::UrlGenerationError)
    ExpectedRoutingError = ActionController::UrlGenerationError
  else
    ExpectedRoutingError = ActionController::RoutingError
  end
  
  RSpec.describe ApplicationController, :type => :controller do
    controller do
      def index
        render :plain => "index called"
      end
  
      def create
        render :plain => "create called"
      end
  
      def new
        render :plain => "new called"
      end
  
      def show
        render :plain => "show called"
      end
  
      def edit
        render :plain => "edit called"
      end
  
      def update
        render :plain => "update called"
      end
  
      def destroy
        render :plain => "destroy called"
      end
  
      def willerror
        render :plain => "will not render"
      end
    end
  
    describe "#index" do
      it "responds to GET" do
        get :index
        expect(response.body).to eq "index called"
      end
  
      it "also responds to POST" do
        post :index
        expect(response.body).to eq "index called"
      end
  
      it "also responds to PUT" do
        put :index
        expect(response.body).to eq "index called"
      end
  
      it "also responds to DELETE" do
        delete :index
        expect(response.body).to eq "index called"
      end
    end
  
    describe "#create" do
      it "responds to POST" do
        post :create
        expect(response.body).to eq "create called"
      end
  
      # And the rest...
      %w{get post put delete}.each do |calltype|
        it "responds to #{calltype}" do
          send(calltype, :create)
          expect(response.body).to eq "create called"
        end
      end
    end
  
    describe "#new" do
      it "responds to GET" do
        get :new
        expect(response.body).to eq "new called"
      end
  
      # And the rest...
      %w{get post put delete}.each do |calltype|
        it "responds to #{calltype}" do
          send(calltype, :new)
          expect(response.body).to eq "new called"
        end
      end
    end
  
    describe "#edit" do
      it "responds to GET" do
        get :edit, :params => { :id => "anyid" }
        expect(response.body).to eq "edit called"
      end
  
      it "requires the :id parameter" do
        expect { get :edit }.to raise_error(ExpectedRoutingError)
      end
  
      # And the rest...
      %w{get post put delete}.each do |calltype|
        it "responds to #{calltype}" do
          send(calltype, :edit, :params => {:id => "anyid"})
          expect(response.body).to eq "edit called"
        end
      end
    end
  
    describe "#show" do
      it "responds to GET" do
        get :show, :params => { :id => "anyid" }
        expect(response.body).to eq "show called"
      end
  
      it "requires the :id parameter" do
        expect { get :show }.to raise_error(ExpectedRoutingError)
      end
  
      # And the rest...
      %w{get post put delete}.each do |calltype|
        it "responds to #{calltype}" do
          send(calltype, :show, :params => {:id => "anyid"})
          expect(response.body).to eq "show called"
        end
      end
    end
  
    describe "#update" do
      it "responds to PUT" do
        put :update, :params => { :id => "anyid" }
        expect(response.body).to eq "update called"
      end
  
      it "requires the :id parameter" do
        expect { put :update }.to raise_error(ExpectedRoutingError)
      end
  
      # And the rest...
      %w{get post put delete}.each do |calltype|
        it "responds to #{calltype}" do
          send(calltype, :update, :params =>  {:id => "anyid"})
          expect(response.body).to eq "update called"
        end
      end
    end
  
    describe "#destroy" do
      it "responds to DELETE" do
        delete :destroy, :params => { :id => "anyid" }
        expect(response.body).to eq "destroy called"
      end
  
      it "requires the :id parameter" do
        expect { delete :destroy }.to raise_error(ExpectedRoutingError)
      end
  
      # And the rest...
      %w{get post put delete}.each do |calltype|
        it "responds to #{calltype}" do
          send(calltype, :destroy, :params => {:id => "anyid"})
          expect(response.body).to eq "destroy called"
        end
      end
    end
  
    describe "#willerror" do
      it "cannot be called" do
        expect { get :willerror }.to raise_error(ExpectedRoutingError)
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 为不从应用程序控制器继承的匿名控制器创建自定义路由

  ```ruby
  # spec/controllers/other_controller_spec.rb
  require "rails_helper"
  class OtherController < ActionController::Base
  end
  
  RSpec.describe OtherController, :type => :controller do
    controller do
      def custom
        render :plain => "custom called"
      end
    end
  
    specify "manually draw the route to request a custom action" do
      routes.draw { get "custom" => "other#custom" }
  
      get :custom
      expect(response.body).to eq "custom called"
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 为定义的控制器创建自定义路由

  ```ruby
  # spec/controllers/application_controller_spec.rb
  require "rails_helper"
  
  class FoosController < ApplicationController; end
  
  RSpec.describe ApplicationController, :type => :controller do
    controller FoosController do
      def custom
        render :plain => "custom called"
      end
    end
  
    specify "manually draw the route to request a custom action" do
      routes.draw { get "custom" => "foos#custom" }
  
      get :custom
      expect(response.body).to eq "custom called"
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 与命名空间控制器一起使用

  ```ruby
  # spec/controllers/namespaced_controller_spec.rb
  require "rails_helper"
  
  class ApplicationController < ActionController::Base; end
  
  module Outer
    module Inner
      class FoosController < ApplicationController; end
    end
  end
  
  RSpec.describe Outer::Inner::FoosController, :type => :controller do
    controller do
      def index
        @name = self.class.name
        @controller_name = controller_name
        render :plain => "Hello World"
      end
    end
  
    it "creates anonymous controller derived from the namespace" do
      expect(controller).to be_a_kind_of(Outer::Inner::FoosController)
    end
  
    it "gets the class name as described" do
      expect{ get :index }.to change{
        assigns[:name]
      }.to eq('Outer::Inner::FoosController')
    end
  
    it "gets the controller_name as described" do
      expect{ get :index }.to change{
        assigns[:controller_name]
      }.to eq('foos')
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 参阅被测控制器中的应用程序路由

  ```ruby
  # spec/controllers/application_controller_spec.rb
  require "rails_helper"
  
  Rails.application.routes.draw do
    match "/login" => "sessions#new", :as => "login", :via => "get"
  end
  
  RSpec.describe ApplicationController, :type => :controller do
    controller do
      def index
        redirect_to login_url
      end
    end
  
    it "redirects to the login page" do
      get :index
      expect(response).to redirect_to("/login")
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试



### 忽略rescue

使用`bypass_rescue`可以绕过两个Rails对错误的默认处理控制器动作，以及使用screw_from声明的任何自定义处理。

这使您可以指定引发异常的详细信息。

#### 背景

```ruby
# spec/controllers/gadgets_controller_spec_context.rb
class AccessDenied < StandardError; end

class ApplicationController < ActionController::Base
  rescue_from AccessDenied, :with => :access_denied

  private

  def access_denied
    redirect_to "/401.html"
  end
end
```

- 使用`rescue_from`处理标准异常

  ```ruby
  # spec/controllers/gadgets_controller_spec.rb
  require "rails_helper"
  
  require 'controllers/gadgets_controller_spec_context'
  
  RSpec.describe GadgetsController, :type => :controller do
    before do
      def controller.index
        raise AccessDenied
      end
    end
  
    describe "index" do
      it "redirects to the /401.html page" do
        get :index
        expect(response).to redirect_to("/401.html")
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/controllers/gadgets_controller_spec.rb
  ```

  all通过测试

- 使用`bypass_rescue`绕过`rescue_from`处理

  ```ruby
  # spec/controllers/gadgets_controller_spec.rb
  require "rails_helper"
  
  require 'controllers/gadgets_controller_spec_context'
  
  RSpec.describe GadgetsController, :type => :controller do
    before do
      def controller.index
        raise AccessDenied
      end
    end
  
    describe "index" do
      it "raises AccessDenied" do
        bypass_rescue
        # 断言抛出异常
        expect { get :index }.to raise_error(AccessDenied)
      end
    end
  end
  ```
  
  运行测试
  
  ```shell
  $ rspec spec/controllers/gadgets_controller_spec.rb
  ```
  
  all通过测试
  
  

### 路由控制

- 指定路由引擎

  ```ruby
  # spec/controllers/widgets_controller_spec.rb
  require "rails_helper"
  
  # A very simple Rails engine
  module MyEngine
    class Engine < ::Rails::Engine
      isolate_namespace MyEngine
    end
  
    Engine.routes.draw do
      resources :widgets, :only => [:show] do
        get :random, :on => :collection
      end
    end
  
    class WidgetsController < ::ActionController::Base
      def random
        @random_widget = Widget.all.shuffle.first
        redirect_to widget_path(@random_widget)
      end
  
      def show
        @widget = Widget.find(params[:id])
        render :text => @widget.name
      end
    end
  end
  
  RSpec.describe MyEngine::WidgetsController, :type => :controller do
    routes { MyEngine::Engine.routes }
  
    it "redirects to a random widget" do
      widget1 = Widget.create!(:name => "Widget 1")
      widget2 = Widget.create!(:name => "Widget 2")
  
      get :random
      expect(response).to be_redirect
      expect(response).to redirect_to(assigns(:random_widget))
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试



### Cookies

- 测试在控制器中清除cookie的值

  ```ruby
  # spec/controllers/application_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe ApplicationController, :type => :controller do
    controller do
      def clear_cookie
        cookies.delete(:user_name)
        head :ok
      end
    end
  
    before do
      routes.draw { get "clear_cookie" => "anonymous#clear_cookie" }
    end
  
    it "clear cookie's value 'user_name'" do
      cookies[:user_name] = "Sam"
  
      get :clear_cookie
  
      expect(cookies[:user_name]).to eq nil
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试



### 设置请求头

不推荐直接在控制器中设置请求头，如果非要设置，请使用下面的 `request.headers` 方法

- 设置请求头

  ```ruby
  # spec/controllers/application_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe ApplicationController, type: :controller do
    controller do
      def show
        if request.headers["Authorization"] == "foo"
          head :ok
        else
          head :forbidden
        end
      end
    end
  
    before do
      routes.draw { get "show" => "anonymous#show" }
    end
  
    context "valid Authorization header" do
      it "returns a 200" do
        request.headers["Authorization"] = "foo"
  
        get :show
  
        expect(response).to have_http_status(:ok)
      end
    end
  
    context "invalid Authorization header" do
      it "returns a 403" do
        request.headers["Authorization"] = "bar"
  
        get :show
  
        expect(response).to have_http_status(:forbidden)
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试



## 期望值匹配

rspec-rails提供了许多自定义匹配器，其中大多数是rspec的用于Rails的断言装饰器。所以完全兼容Rails默认的断言方式。

关于兼容性，例如：

```ruby
# rails 写法
assert_equal 403, response.status
assert_response 403
# rspec写法
expect(response).to have_http_status(403)
```

完全等价

### Be_a_new 新建

- 四个be_a_new 示例

  ```ruby
  # spec/models/widget_spec.rb
  require "rails_helper"
  
  RSpec.describe Widget do
    context "when initialized" do
      subject(:widget) { Widget.new }
  
      it "is a new widget" do
        expect(widget).to be_a_new(Widget)
      end
  
      it "is not a new string" do
        expect(widget).not_to be_a_new(String)
      end
    end
  
    context "when saved" do
      subject(:widget) { Widget.create }
  
      it "is not a new widget" do
        expect(widget).not_to be_a_new(Widget)
      end
  
      it "is not a new string" do
        expect(widget).not_to be_a_new(String)
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/models/widget_spec.rb
  ```

  all通过测试

### Render_template

- 3种 render_template 示例

  ```ruby
  # spec/controllers/gadgets_spec.rb
  require "rails_helper"
  
  RSpec.describe GadgetsController do
    describe "GET #index" do
      subject { get :index }
  
      it "renders the index template" do
        expect(subject).to render_template(:index)
        expect(subject).to render_template("index")
        expect(subject).to render_template("gadgets/index")
      end
  
      it "does not render a different template" do
        expect(subject).to_not render_template("gadgets/show")
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/controllers/gadgets_spec.rb
  ```

  all通过测试

- 指定layout

  ```ruby
  # spec/controllers/gadgets_spec.rb
  require "rails_helper"
  
  RSpec.describe GadgetsController do
    describe "GET #index" do
      subject { get :index }
  
      it "renders the application layout" do
        expect(subject).to render_template("layouts/application")
      end
  
      it "does not render a different layout" do
        expect(subject).to_not render_template("layouts/admin")
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/controllers/gadgets_spec.rb
  ```

  all通过测试

- 视图 中使用

  ```ruby
  # spec/views/gadgets/index.html.erb_spec.rb
  require "rails_helper"
  
  RSpec.describe "gadgets/index" do
    it "renders the index template" do
      assign(:gadgets, [Gadget.create!])
      render
  
      expect(view).to render_template(:index)
      expect(view).to render_template("index")
      expect(view).to render_template("gadgets/index")
    end
  
    it "does not render a different template" do
      expect(view).to_not render_template("gadgets/show")
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/views
  ```

  all通过测试



### Redirect_to

- 4种 redirect_to示例

  ```ruby
  # spec/controllers/widgets_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe WidgetsController do
  
    describe "#create" do
      subject { post :create, :params => { :widget => { :name => "Foo" } } }
  
      it "redirects to widget_url(@widget)" do
        expect(subject).to redirect_to(widget_url(assigns(:widget)))
      end
  
      it "redirects_to :action => :show" do
        expect(subject).to redirect_to :action => :show,
                                       :id => assigns(:widget).id
      end
  
      it "redirects_to(@widget)" do
        expect(subject).to redirect_to(assigns(:widget))
      end
  
      it "redirects_to /widgets/:id" do
        expect(subject).to redirect_to("/widgets/#{assigns(:widget).id}")
      end
    end
  end
  ```

  运行测试

  ```shell
  $  rspec spec/controllers/widgets_controller_spec.rb
  ```

  all通过测试



### have_http_status

- 检查数字状态码

  ```ruby
  # spec/controllers/application_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe ApplicationController, :type => :controller do
  
    controller do
      def index
        render :json => {}, :status => 209
      end
    end
  
    describe "GET #index" do
      it "returns a 209 custom status code" do
        get :index
        expect(response).to have_http_status(209)
      end
    end
  
  end
  ```

  运行测试

  ```shell
  $  rspec spec
  ```

  all通过测试

- 检查符号状态码

  ```ruby
  # spec/controllers/application_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe ApplicationController, :type => :controller do
  
    controller do
      def index
        render :json => {}, :status => :see_other
      end
    end
  
    describe "GET #index" do
      it "returns a :see_other status code" do
        get :index
        expect(response).to have_http_status(:see_other)
      end
    end
  
  end
  ```

  运行测试

  ```shell
  $  rspec spec
  ```

  all通过测试

- 检查通用符号状态类型

  ```ruby
  # spec/controllers/application_controller_spec.rb
  require "rails_helper"
  
  RSpec.describe ApplicationController, :type => :controller do
  
    controller do
      def index
        render :json => {}, :status => :bad_gateway
      end
    end
  
    describe "GET #index" do
      it "returns a some type of error status code" do
        get :index
        expect(response).to have_http_status(:error)
      end
    end
  
  end
  ```

  运行测试

  ```shell
  $  rspec spec
  ```

  all通过测试

- 在控制器中使用

  ```ruby
  # spec/controllers/gadgets_spec.rb
  require "rails_helper"
  
  RSpec.describe GadgetsController, :type => :controller do
  
    describe "GET #index" do
      it "returns a 200 OK status" do
        get :index
        expect(response).to have_http_status(:ok)
      end
    end
  
  end
  ```

  运行测试

  ```shell
  $  rspec spec/controllers/gadgets_spec.rb
  ```

  all通过测试

- 在request中使用

  ```ruby
  # spec/requests/gadgets/widget_management_spec.rb
  require "rails_helper"
  
  RSpec.describe "Widget management", :type => :request do
  
    it "creates a Widget and redirects to the Widget's page" do
      get "/widgets/new"
      expect(response).to have_http_status(:ok)
  
      post "/widgets", :params => { :widget => {:name => "My Widget"} }
      expect(response).to have_http_status(302)
  
      follow_redirect!
  
      expect(response).to have_http_status(:success)
    end
  
  end
  ```

  运行测试

  ```shell
  $  rspec spec/requests
  ```

  all通过测试

- 在 feature 中使用

  ```ruby
  # spec/features/widget_management_spec.rb
  require "rails_helper"
  
  RSpec.feature "Widget management", :type => :feature do
  
    scenario "User creates a new widget" do
      visit "/widgets/new"
      expect(page).to have_http_status(200)
  
      click_button "Create Widget"
  
      expect(page).to have_http_status(:success)
    end
  
  end
  ```

  运行测试

  ```shell
  $  rspec spec/features/widget_management_spec.rb
  ```

  all通过测试



### match_array

- 实例

  ```ruby
  # spec/models/widget_spec.rb
  require "rails_helper"
  
  RSpec.describe Widget do
    let!(:widgets) { Array.new(3) { Widget.create } }
  
    subject { Widget.all }
  
    it "returns all widgets in any order" do
      expect(subject).to match_array(widgets)
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/models/widget_spec.rb
  ```

  all通过测试



### Have_been_enqueued

检查指定的任务是否在排队

- 检查任务的类名

  ```ruby
  # spec/jobs/upload_backups_job_spec.rb
  require "rails_helper"
  
  RSpec.describe UploadBackupsJob do
    it "matches with enqueued job" do
      ActiveJob::Base.queue_adapter = :test
      UploadBackupsJob.perform_later
      expect(UploadBackupsJob).to have_been_enqueued
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/jobs/upload_backups_job_spec.rb
  ```

  all通过测试

- 检查任务传参

  ```ruby
  # spec/jobs/upload_backups_job_spec.rb
  require "rails_helper"
  
  RSpec.describe UploadBackupsJob do
    it "matches with enqueued job" do
      ActiveJob::Base.queue_adapter = :test
      UploadBackupsJob.perform_later("users-backup.txt", "products-backup.txt")
      expect(UploadBackupsJob).to(
        have_been_enqueued.with("users-backup.txt", "products-backup.txt")
      )
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/jobs/upload_backups_job_spec.rb
  ```

  all通过测试

- 检查任务排队时间

  ```ruby
  # spec/jobs/upload_backups_job_spec.rb"
  require "rails_helper"
  
  RSpec.describe UploadBackupsJob do
    it "matches with enqueued job" do
      ActiveJob::Base.queue_adapter = :test
      UploadBackupsJob.set(:wait_until => Date.tomorrow.noon).perform_later
      expect(UploadBackupsJob).to have_been_enqueued.at(Date.tomorrow.noon)
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/jobs/upload_backups_job_spec.rb
  ```

  all通过测试

- 检查没有等待的工作

  ```ruby
  # spec/jobs/upload_backups_job_spec.rb
  require "rails_helper"
  
  RSpec.describe UploadBackupsJob do
    it "matches with enqueued job" do
      ActiveJob::Base.queue_adapter = :test
      UploadBackupsJob.perform_later
      expect(UploadBackupsJob).to have_been_enqueued.at(:no_wait)
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/jobs/upload_backups_job_spec.rb
  ```

  all通过测试

- 检查任务队列名称

  ```ruby
  # spec/jobs/upload_backups_job_spec.rb" with:
  require "rails_helper"
  
  RSpec.describe UploadBackupsJob do
    it "matches with enqueued job" do
      ActiveJob::Base.queue_adapter = :test
      UploadBackupsJob.perform_later
      expect(UploadBackupsJob).to have_been_enqueued.on_queue("default")
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/jobs/upload_backups_job_spec.rb
  ```

  all通过测试



### Have_been_performed

检查任务是否已经执行

- 检查任务类名

  ```ruby
  # spec/jobs/upload_backups_job_spec.rb
  require "rails_helper"
  
  RSpec.describe UploadBackupsJob do
    it "matches with performed job" do
      ActiveJob::Base.queue_adapter = :test
      ActiveJob::Base.queue_adapter.perform_enqueued_jobs = true
      UploadBackupsJob.perform_later
      expect(UploadBackupsJob).to have_been_performed
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/jobs/upload_backups_job_spec.rb
  ```

  all通过测试

- 检查任务参数

  ```ruby
  # spec/jobs/upload_backups_job_spec.rb
  require "rails_helper"
  
  RSpec.describe UploadBackupsJob do
    it "matches with performed job" do
      ActiveJob::Base.queue_adapter = :test
      ActiveJob::Base.queue_adapter.perform_enqueued_jobs = true
      UploadBackupsJob.perform_later("users-backup.txt", "products-backup.txt")
      expect(UploadBackupsJob).to(
        have_been_performed.with("users-backup.txt", "products-backup.txt")
      )
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/jobs/upload_backups_job_spec.rb
  ```

  all通过测试

- 检查任务执行时间

  ```ruby
  # spec/jobs/upload_backups_job_spec.rb
  require "rails_helper"
  
  RSpec.describe UploadBackupsJob do
    it "matches with performed job" do
      ActiveJob::Base.queue_adapter = :test
      ActiveJob::Base.queue_adapter.perform_enqueued_jobs = true
      ActiveJob::Base.queue_adapter.perform_enqueued_at_jobs = true
      UploadBackupsJob.set(:wait_until => Date.tomorrow.noon).perform_later
      expect(UploadBackupsJob).to have_been_performed.at(Date.tomorrow.noon)
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/jobs/upload_backups_job_spec.rb
  ```

  all通过测试

  

-  检查任务队列名

  ```ruby
  # spec/jobs/upload_backups_job_spec.rb
  require "rails_helper"
  
  RSpec.describe UploadBackupsJob do
    it "matches with performed job" do
      ActiveJob::Base.queue_adapter = :test
      ActiveJob::Base.queue_adapter.perform_enqueued_jobs = true
      UploadBackupsJob.perform_later
      expect(UploadBackupsJob).to have_been_performed.on_queue("default")
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/jobs/upload_backups_job_spec.rb
  ```

  all通过测试



## request rspec

### Request spec

- 使用Rails内部方法管理 Widget

  ```ruby
  # spec/requests/widget_management_spec.rb
  require "rails_helper"
  
  RSpec.describe "Widget management", :type => :request do
  
    it "creates a Widget and redirects to the Widget's page" do
      get "/widgets/new"
      expect(response).to render_template(:new)
  
      post "/widgets", :params => { :widget => {:name => "My Widget"} }
  
      expect(response).to redirect_to(assigns(:widget))
      follow_redirect!
  
      expect(response).to render_template(:show)
      expect(response.body).to include("Widget was successfully created.")
    end
  
    it "does not render a different template" do
      get "/widgets/new"
      expect(response).to_not render_template(:show)
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/requests/widget_management_spec.rb
  ```

  all通过测试

- 发送json请求数据

  ```ruby
  # spec/requests/widget_management_spec.rb
  require "rails_helper"
  
  RSpec.describe "Widget management", :type => :request do
  
    it "creates a Widget" do
      # 设置请求头
      headers = { "ACCEPT" => "application/json" }
      # 设置响应数据格式
      post "/widgets", :params => { :widget => {:name => "My Widget"} }, :headers => headers
  
      expect(response.content_type).to eq("application/json")
      expect(response).to have_http_status(:created)
    end
  
  end
  ```

  运行测试

  ```shell
  $ rspec spec/requests/widget_management_spec.rb
  ```

  all通过测试

- 请求json响应数据

  ```ruby
  # spec/requests/widget_management_spec.rb
  require "rails_helper"
  
  RSpec.describe "Widget management", :type => :request do
    it "creates a Widget" do
      headers = { "ACCEPT" => "application/json" }
      post "/widgets", :params => { :widget => {:name => "My Widget"} }, :headers => headers
  
      expect(response.content_type).to eq("application/json; charset=utf-8")
      expect(response).to have_http_status(:created)
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/requests/widget_management_spec.rb
  ```

  all通过测试

- 请求json数据

  ```ruby
  # spec/requests/widget_management_spec.rb
  require "rails_helper"
  
  RSpec.describe "Widget management", :type => :request do
  
    it "creates a Widget and redirects to the Widget's page" do
      # 设置请求数据的格式
      headers = { "CONTENT_TYPE" => "application/json" }
      post "/widgets", :params => '{ "widget": { "name":"My Widget" } }', :headers => headers
      expect(response).to redirect_to(assigns(:widget))
    end
  
  end
  ```

  运行测试

  ```shell
  $ rspec spec/requests/widget_management_spec.rb
  ```

  all通过测试

- 使用路由辅助方法

  ```ruby
  # spec/requests/widgets_spec.rb
  require "rails_helper"
  
  # 简单的 Rails engine
  module MyEngine
    class Engine < ::Rails::Engine
      isolate_namespace MyEngine
    end
  
    class LinksController < ::ActionController::Base
      def index
        render plain: 'hit_engine_route'
      end
    end
  end
  
  MyEngine::Engine.routes.draw do
    resources :links, :only => [:index]
  end
  
  Rails.application.routes.draw do
    mount MyEngine::Engine => "/my_engine"
  end
  
  module MyEngine
    RSpec.describe "Links", :type => :request do
      include Engine.routes.url_helpers
  
      it "redirects to a random widget" do
        # 允许使用 辅助方法访问
        get links_path
        expect(response.body).to eq('hit_engine_route')
      end
    end
  end
  ```
  
  运行测试
  
  ```shell
  $ rspec spec
  ```
  
  all通过测试



## Mailer specs

### URL helpers

- 使用url辅助方法的默认参数

  ```ruby
  # config/initializers/mailer_defaults.rb
  Rails.configuration.action_mailer.default_url_options = { :host => 'example.com' }
  
  
  # spec/mailers/notifications_spec.rb
  require 'rails_helper'
  
  RSpec.describe NotificationsMailer, :type => :mailer do
    it 'should have access to URL helpers' do
      expect { gadgets_url }.not_to raise_error
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试

- 使用url辅助方法,不使用默认参数

  ```ruby
  # config/initializers/mailer_defaults.rb
  # no default options
  
  #spec/mailers/notifications_spec.rb
  require 'rails_helper'
  
  RSpec.describe NotificationsMailer, :type => :mailer do
    it 'should have access to URL helpers' do
      expect { gadgets_url :host => 'example.com' }.not_to raise_error
      expect { gadgets_url }.to raise_error
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试



### Mailer

- 简单实例

  ```ruby
  # spec/mailers/notifications_mailer_spec.rb" with:
  require "rails_helper"
  
  # 最外层的架构快
  # RSpec.describe 类名, :type => :类型 do
  RSpec.describe NotificationsMailer, :type => :mailer do
    # 一个 describe 块
    describe "notify" do
      let(:mail) { NotificationsMailer.signup }
  	# 一个测试用例
      it "renders the headers" do
        # expect(值).to eq(等于的值)
        expect(mail.subject).to eq("Signup")
        expect(mail.to).to eq(["to@example.org"])
        expect(mail.from).to eq(["from@example.com"])
      end
  	# 一个测试用例
      it "renders the body" do
        expect(mail.body.encoded).to match("Hi")
      end
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec
  ```

  all通过测试



## File fixture

Rails 5 新增 file fixture 对象。

稳进默认存储路径：spec/fixtures/files

File fixtures 代表了 +Pathname+ 对象，使用很简单。

实例：

```
file_fixture("example.txt").read # get the file's content
file_fixture("example.mp3").size # get the file size
```

通过配置可以改变默认文件位置

```
RSpec.configure do |config|
  config.file_fixture_path = "spec/custom_directory"
end
```



- 读取文件内容

  ```ruby
  # spec/fixtures/files/sample.txt
  Hello
  
  
  # spec/lib/file_spec.rb
  require "rails_helper"
  
  RSpec.describe "file" do
    it "reads sample file" do
      #期待.读取文件的内容.为 等于("hello")
      expect(file_fixture("sample.txt").read).to eq("Hello")
    end
  end
  ```

  运行测试

  ```shell
  $ rspec spec/lib/file_spec.rb
  ```

  all通过测试

