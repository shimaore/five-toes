    class Content
      constructor: (@event,@content_type,body) ->
        @body = Buffer.from body

    module.exports = Content
