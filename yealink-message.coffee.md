    Content = require './content'

    module.exports = yealink_message = (message,beep,timeout) ->
      beep ?= no
      timeout ?= 900
      new Content 'Yealink-xml', 'application/xml', """
        <?xml version="1.0"?>
        <YealinkIPPhoneStatus Beep="#{if beep then 'yes' else 'no'}" Timeout="#{timeout}">
        <Message Align="center">#{message}</Message>
        </YealinkIPPhoneStatus>
      """
