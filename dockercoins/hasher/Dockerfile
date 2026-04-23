FROM ruby:alpine
RUN apk add --update build-base curl
RUN gem install sinatra -v 3.1.0
RUN gem install thin
ADD hasher.rb /
CMD ["ruby", "hasher.rb"]
EXPOSE 80
