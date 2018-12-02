---
layout: post
title: "devise + omniauth-facebook でアクセストークンを取得する方法"
date: 2012-03-04 18:02:00 +09:00
tags: web
image: /images/profile.png
---

Facebook に devise + omniauth-facebook で OAuth2 認証した時のアクセストークンの取得方法を備忘録がてらメモ。環境は Rails 3.1.1, devise 1.5.2, omniauth-facebook 1.1.0。

アクセストークンはコールバックアクションで取得できる。[OmniAuth  Overview · plataformatec/devise Wiki](https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview "OmniAuth Overview · plataformate/deise Wiki") のインストラクションに従っているなら、RAILS_ROOT/app/controllers/user/omniauth_callbacks_controller.rb 内の `request.env["omniauth.auth"].credentials.token` がそれ。

```ruby
class User::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def facebook
    # You need to implement the method below in your model
    @user = User.find_for_facebook_oauth(request.env["omniauth.auth"], current_user)

    if @user.persisted?
      flash[:notice] = I18n.t "devise.omniauth_callbacks.success", :kind => "Facebook"
      sign_in_and_redirect @user, :event => :authentication
    else
      session["devise.facebook_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end
end
```

`User.find_for_facebook_oauth` 内で保存して、Facebook API を叩く時に使う。なお、API を叩く時は fb_graph を使ってます。
