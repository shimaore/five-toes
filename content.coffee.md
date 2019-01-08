    class Content
      constructor: (@event,@type,body) ->
        @body = Buffer.from body

    module.exports = Content
