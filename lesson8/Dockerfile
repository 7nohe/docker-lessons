FROM ruby:2.5
# 必要なパッケージのインストール
RUN apt-get update -qq && apt-get install -y nodejs postgresql-client
RUN mkdir /rails_app
WORKDIR /rails_app
COPY Gemfile /rails_app/Gemfile
COPY Gemfile.lock /rails_app/Gemfile.lock
RUN bundle install
COPY . /rails_app

# コンテナ起動時に実行されるスクリプトを配置
COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 3000

# Railsサーバーを起動
CMD ["rails", "server", "-b", "0.0.0.0"]
