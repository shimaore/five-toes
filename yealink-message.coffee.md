    Content = require './content'

    module.exports = yealink_message = (message) ->
      new Content 'Yealink-xml', 'application/xml', """
        <?xml version="1.0"?>
        <YealinkIPPhoneStatus Beep="yes" Timeout="900">
        <Message>#{message}</Message>
        </YealinkIPPhoneStatus>
      """
