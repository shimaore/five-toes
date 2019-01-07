    Content = require './content'

    module.exports = message_summary = (new_messages,saved_messages) ->

[RFC3842](https://tools.ietf.org/html/rfc3842)

      new Content 'message-summary', 'application/simple-message-summary', """
        Message-Waiting: #{if new_messages > 0 then 'yes' else 'no'}
        Voice-Message: #{new_messages}/#{saved_messages}

      """
