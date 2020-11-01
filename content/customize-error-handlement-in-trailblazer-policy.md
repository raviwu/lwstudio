+++
title = "在 Trailblazer 的 Policy 中透過客制 Exception 來處理複雜的「錯誤回應」"
date = 2017-01-14T13:25:31+08:00
publishdate = 2017-01-14T13:25:31+08:00
tags = ["coding", "rails", "ruby", "trailblazer"]
draft = false

description = ""
summary = ""
keywords = []

[amp]
    elements = []

[author]
    name = "Ravi Wu"
    homepage = "https://raviwu.github.io"

[image]
    src = ""

[sitemap]
    changefreq = "monthly"
    priority = 0.5
    filename = "sitemap.xml"
+++

為了盡量貼近 Trailblazer 的設計概念，許多原先會透過 Controller `before_action` 去處理的權限管控，盡量都搬進 Trailblazer 的 [Policy](http://trailblazer.to/gems/operation/1.1/policy.html) 裡。

Policy 採用類似 [Pundit](https://github.com/elabs/pundit) 的語法，典型的 Policy 如下：

```ruby
class Thing::Policy
    def initialize(user, thing)
    @user, @thing = user, thing
    end

    def create?
    (admin? || approved?) && @thing.persisted?
    end

    private

    def admin?
    @user.admin == true
    end

    def approved?
    @user.is_approved
    end
end
```

在 Operation 中若要調用這隻 Policy 的話要宣告：

```ruby
class Thing::Create < Trailblazer::Operation
    builds -> (params) { dispatched_class_accroding_to(params) }

    def self.dispatched_class_accroding_to(params)
    Thing::Create
    end

    include Trailblazer::Operation::Policy
    policy Thing::Policy, :create?

    def process(params)
    validate(params[:thing]) do |f|
        f.save
    end
    end
end
```

Operation 在 controller 裡被 `call` 的時候，會先跑 `builds` 去回傳 Operation 類別，此時，正是個機會去依照 params 內容 dispatch 到其他的 Operation 去，改天再分享利用 `builds` 簡化路由的實作。

透過 builds 確認 operation 後，所有的參數會先經過 `setup!(params)` > `setup_params!(params)` > `model!(params)` > `setup_model!` 等 operation 的 method 去建立這隻 operation 操作所需的背景條件（基本上都是要依照條件去撈 model 出來，或是要對參數做前處理等，詳細 callstack 請參考這個[頁面](http://trailblazer.to/gems/operation/1.1/api.html)），跑完整個 setup 的過程後，如果有指定 Policy 才會送進 Policy 的對應 method 裡。

Policy initialize 裡的 `user` 跟 `thing` 對應到 `params[:current_user]` 跟 `operation.model` 這兩個物件，前述的 `params[:current_user]` 如果跑完 `setup!(params)` > `setup_params!(params)` > `model!(params)` > `setup_model!` 都沒有指定的話會預設 `nil` 值，而 `operation.model` 則是 `model!(params)` > `setup_model!` 中所指定的 resource。如果 `user` / `thing` 的狀態在 `Thing::Policy#create?` 裡的運算結果為 `true` 時，才會繼續送到 `process(params)` 的方法中繼續處理，不然就會丟 `Trailblazer::NotAuthorizedError` 例外。

問題來了，在 controller 裡面，案例給的很乾淨：

```ruby
class CommentsController < ApplicationController
    def create
    run Comment::Create do |op|
        flash[:notice] = "Success!" # only run for successful/valid operation.
        return redirect_to thing_path(@model.thing)
    end

    render :new
    end
end
```

但如果需要針對不同的 `Trailblazer::NotAuthorizedError` 情況來轉址或者是設定錯誤訊息，在只使用 `Trailblazer::NotAuthorizedError` 的情況下，變成要在 controller 端另外加判斷，例如：

```ruby
class CommentsController < ApplicationController
    def create
    run Comment::Create do |op|
        flash[:notice] = "Success!" # only run for successful/valid operation.
        return redirect_to thing_path(@model.thing)
    end

    render :new
    rescue Trailblazer::NotAuthorizedError
    flash[:alert] = "You're not Admin!" unless @user.admin
    flash[:alert] = "Your account is not approved!" unless @user.is_approved

    redirect_to root_path
    end
end
```

安捏很奇怪，因為權限在 Policy 裡面判斷了一次，然後跑到 controller 裡又得再重新跑一次幾乎一樣的判斷，目前暫時的解法，是在 Policy 裡面加一個 custom error 讓 controller 統一處理需要特別關照的 `Trailblazer::NotAuthorizedError` 狀況。

```ruby
module Thing
    class Policy
    def initialize(user, thing)
        @user, @thing = user, thing
    end

    def create?
        raise Thing::NotAuthorizedError, "You're not Admin!" unless admin?
        raise Thing::NotAuthorizedError, "Your account is not approved!" unless approved?
        @thing.persisted?
    end

    private

    def admin?
        @user.admin == true
    end

    def approved?
        @user.is_approved
    end
    end

    class NotAuthorizedError < StandardError; end
end

# in controller
class CommentsController < ApplicationController
    def create
    run Comment::Create do |op|
        flash[:notice] = "Success!" # only run for successful/valid operation.
        return redirect_to thing_path(@model.thing)
    end

    render :new
    rescue Thing::NotAuthorizedError => e
    flash[:alert] = e.message
    redirect_to root_path
    rescue Trailblazer::NotAuthorizedError
    flash[:alert] = "Action not authorized."
    redirect_to root_path
    end
end
```

改成這樣後，雖然 controller 還是對於例外細節如何處理仍有耦合，但至少邏輯判斷不再在 Policy / Controller 裡重複，然後也能進一步將一般性的 `Trailblazer::NotAuthorizedError` 抽到 ApplicationController 裡面統一處理，對於一些複雜的例外處理需求，目前這麼做還算暫時可以接受，如果你有發現其他更好的方式，跪求指教。