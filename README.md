> #### 1. 게시판 만들기 !
>
> ```
> # Board 만들기
> # 1. get '/' {} : index.erb
> # => 지금까지 써진 모든 글들을 보여준다.
> # => '글쓰기' 링크가 있고 -> '/new'
> # 2. get '/new' : new.erb
> # => 새로운 글을 쓸 수 있는 <form>, title, content -> /'create'
> # 3. get '/create' :
> # => new에서 보내준 정보를 바탕으로 Post.create()
> # => redirect -> '/'
> # 4. 회원가입 '/signup'
> # 5. 로그인 '/login'
> ```
>
> - **app.rb 일부분**
>
> ```
> get '/create' do
>  Post.create(
>    :title => params["title"],
>    :content => params["content"],
>    :author => params["author"]
>  )
>
>  redirect to '/'
> end
>
> get '/register' do
>  User.create(
>    :email => params["email"],
>    :password => params["password"]
>  )
> end
>
> get '/login_session' do
>  @message = ""
>
>  if User.first(:email => params["email"])
>    if User.first(:email => params["email"]).password == params["password"]
>      session[:email] = params["email"]
>
>      redirect to '/'
>    else
>      @message = "비밀번호가 틀렸습니다."
>    end
>  else
>    @message = "해당하는 유저의 이메일이 없습니다."
>  end
> end
> ```
>
> - model.rb
>
> ```
> # need install dm-sqlite-adapter
> DataMapper::setup(:default, "sqlite3://#{Dir.pwd}/board.db")
>
> class Post
>  ##DataMapper 객체로 Question를 만들겠다.
>  include DataMapper::Resource
>  property :id, Serial
>  property :title, String
>  property :content, Text
>  property :author, String, :default => "익명"
>  property :created_at, DateTime
> end
>
> class User
>  include DataMapper::Resource
>  property :id, Serial
>  property :email, String
>  property :password, String
>  property :created_at, DateTime
> end
>
> DataMapper.finalize
>
> Post.auto_upgrade!
> User.auto_upgrade!
> ```
>
> - index.erb
>
> ```
> <body>
>  <% @list.each do |q|%>
>    <div class="card">
>      <div class="card-body">
>        <p>번호 : <%= q.id %></p>
>        <p>제목 : <%= q.title %></p>
>        <p>내용 : <%= q.content %></p>
>        <p>작성자 : <%= q.author %></p>
>        <p>작성시간 : <%= q.created_at %></p>
>        <a href="/destroy">[삭제]</a>
>      </div>
>    </div>
>  <% end %>
> </body>
> ```

### 2. CRUD 에서 U, D 를 배워보자 !

> #### 1. datamapper destroy 하는 방법 !
>
> ```
> zoo = Zoo.get(5)
> zoo.destroy  # => true
> ```
>
> **그런데 input으로 값을 받아온게 아닌데 !!!! 어떻게 get() 으로 지우고 싶은 글을 가져올 수 있을까 ?**
>
> #### 어떻게 하느냐 ! variable routing 을 이용해서 임시변수를 만들면 된다 !
>
> ```
> # variable routing
> # hello 뒤에 아무 글자들이 들어와도
> # hello 페이지로 보낸다.
> # 즉, 임의의 변수를 만들어주고
> # 입력된 어떠한 값들을 받아 임시 변수로 만들어 사용할 수 있게끔 만들어줌.
> get '/hello/:name' do
>  @name = params[:name]
>
>  erb :hello
> end
> # # /sqaure/3을 넣으면
> # # sqare.erb 보내주고, 3 * 3 = 9 을 출력
> get '/square/:num' do
>  num = params[:num].to_i
>  # @result = num * num
>  @result = num ** 2
>  erb :square
> end
>
> get '/cube/:num' do
>
> end
> ```
>
> #### 2. 그러면 get '/destroy/:id'를 만들어보자
>
> ```
> get '/destroy/:id' do
>  # 1번 글을 지우게 될거면
>  # '/destroy/1...5'
>  post = Post.get(params[:id])
>  post.destroy
>
>  redirect to '/'
> end
> ```
>
> #### 3. 그러면 이제는 update를 배워보자
>
> - 수정은 폼이 두개가 필요하다 !
> - 먼저 edit으로 정보를 가져온 후 글을 수정한다 ! 이후 update를 통해 수정 된 정보들을 저장한다.
>
> ```
> get '/edit/:id' do
>  # 수정하기 위해선
>  # 수정하고 싶은 정보가 edit에 넘어와야 한다 !
>  # 그러니 여기에서도 variable routing을 이용한다.
>  @post = Post.get(params[:id])
>
>  erb :edit
> end
>
> get '/update/:id' do
>  post = Post.get(params[:id])
>  post.update(
>    # :title => params["title"], 이렇게 해두 됨ㅋ
>    :title => params[:title],
>    :content => params[:content]
>  )
>
>  redirect to '/'
> end
> ```
>
> - edit.erb
>
> ```
> <body>
>  <h1>수정하기</h1>
>  <form action="/update/<%= @post.id %>">
>      <div class="form-group col-md-4">
>        <input type="text" class="form-control" name="title" value="<%= @post.title %>">
>        <input type="text" class="form-control" name="content" value="<%= @post.content %>">
>        <input type="hidden" name="author" value="">
>      </div>
>      <button type="submit" class="btn btn-primary">Submit</button>
>  </form>
> </body>
> ```

### 3. 심볼과 String의 차이

> 즉, 심볼도 객체이며, 문자열도 객체입니다. 그러나 symbol은 변경이 불가능(immutable)한 객체입니다. 다시말해, 심볼은 한번 값이 assign 되고 나면 값을 변경하는 것이 불가능합니다. 그렇다고 해서 객체자체가 Java의 final 선언된 변수와 같이 덮어쓸 수도 없다는 의미는 아닙니다. Immutable 이란, 객체가 가지고 있는 값을 변경(change)할 수는 없지만 덮어쓰기(overwrite)할 수는 있다는 의미입니다.
>
> 문자열은 변경이 가능(mutable) 하기 때문에, 루비 인터프리터는 실제 해당 문자열이 어떤 값을 가지고 있는지 실행시점까지 알 수 가 없습니다. 이것은 다시말해, 우리가 보기에는 동일한 문자열도 서로 다른 메모리 공간을 차지하고 있어야 한다는 의미입니다.
>
> ```
> [1] pry(main)> :hello
> => :hello
> [2] pry(main)> name ="john"
> => "john"
> [3] pry(main)> "hello #{name}"
> => "hello john"
> [4] pry(main)> name = :jonn
> => :jonn
> [5] pry(main)> "hello #{name}"
> => "hello jonn"
> [6] pry(main)> "jonn" == :jonn
> => false
> [7] pry(main)> "jonn" == "jonn"
> => true
> [8] pry(main)> :jonn == :jonn
> => true
> #### 우리가 보기에는 동일한 문자열도 서로 다른 메모리 공간을 차지하고 있어야 한다는 의미입니다. 아래와 같이 문자열을 생성하고 문자열의 객체ID를 출력해보면 이를 확인할 수 있습니다.
> [11] pry(main)> "john".object_id
> => 47037357059680
> [12] pry(main)> "john".object_id
> => 47037356061040
> [13] pry(main)> "john".object_id
> => 47037359909820
> [14] pry(main)> :john.object_id
> => 1784348
> [15] pry(main)> :john.object_id
> => 1784348
> #### 심볼은 immutable 하기 때문에 한번 heap 메모리상에 생성되고 나면 해당 심볼은 동일한 객체로 재사용이 가능(reusable)합니다.
> [19] pry(main)> hash ={}
> => {}
> [20] pry(main)> hash[:key] = "value"
> => "value"
> [21] pry(main)> hash["key"] = "value new"
> => "value new"
> [22] pry(main)> "key" == :key
> => false
> #
> [23] pry(main)> hash ={key: "value"}
> => {:key=>"value"}
> [24] pry(main)> hash = {:key => "value"}
> => {:key=>"value"}
> # 이 두개는 똑같은 문법. 
>
> ```
>
> 좀더 자세히 설명하자면, 루비에서 심볼은 단순히 동일한 heap 메모리를 재사용할 뿐만 아니라, Symbol dictionary를 통해 관리됩니다. 아래와 같은 명령어를 irb에서 실행하면 현재 Symbol dictionary에 존재하는 심볼 목록을 확인할 수 있습니다.
>
> # 결론
>
> 루비에서 대부분의 경우 심볼을 문자열을 사용하는 경우보다 메모리 효율성이나 성능 측면에서 유리합니다. 이러한 이유로 hash의 키 등으로 문자열을 사용하는 것보다 심볼을 사용하는 것이 좋습니다.
>
> 대부분의 루비 개발자들은 해시의 키로 심볼을 사용하는 것이 익숙하지만, 왜 그렇게 해야하는지, 문자열 대신 심볼을 사용하는 것이 어떤한 면에서 유리한지는 한번 기억해 두는 것이 좋습니다.