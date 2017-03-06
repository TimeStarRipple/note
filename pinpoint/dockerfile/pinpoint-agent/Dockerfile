# DESCRIPTION       Pinpoint APM Agent Volume
# TO_BUILD          docker build -t registry.saas.hand-china.com/tools/pinpoint-agent .
# TO_RUN            docker run --name=pinpoint-agent registry.saas.hand-china.com/tools/pinpoint-agent

FROM registry.saas.hand-china.com/tools/java:8
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
ADD pinpoint-agent-1.6.0.tar /
ADD pinpoint.config /pinpoint-agent-1.6.0/pinpoint.config
