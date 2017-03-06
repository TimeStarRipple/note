# DESCRIPTION       Pinpoint APM Web (User Interface)
# TO_BUILD          docker build -t registry.saas.hand-china.com/tools/pinpoint-web-alone .
# TO_RUN            docker run -d -p 8080:8080 --net=host -e HBASE_HOST=<HOST_IP> -e HBASE_PORT=2181 --name=pinpoint-web registry.saas.hand-china.com/tools/pinpoint-web-alone

FROM tomcat:8.5.2-jre8

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ENV PINPOINT_VERSION=1.6.0

ADD start-web.sh /usr/local/bin/

RUN chmod a+x /usr/local/bin/start-web.sh \
    && mkdir -p /assets/ \
    && curl -SL https://raw.githubusercontent.com/naver/pinpoint/$PINPOINT_VERSION/web/src/main/resources/pinpoint-web.properties -o /assets/pinpoint-web.properties \
    && curl -SL https://raw.githubusercontent.com/naver/pinpoint/$PINPOINT_VERSION/web/src/main/resources/hbase.properties -o /assets/hbase.properties \
    && curl -SL https://github.com/naver/pinpoint/releases/download/$PINPOINT_VERSION/pinpoint-web-$PINPOINT_VERSION.war -o pinpoint-web.war \
    && rm -rf /usr/local/tomcat/webapps \
    && mkdir -p /usr/local/tomcat/webapps \
    && unzip pinpoint-web.war -d /usr/local/tomcat/webapps/ROOT \
    && rm -rf pinpoint-web.war

EXPOSE 8080

ENTRYPOINT ["/usr/local/bin/start-web.sh"]

