Notify a specific URI
=====================

    {debug,foot} = (require 'tangible') "five-toes:send"

    resolve = require './resolve'

Send SIP packet to an URI at a given address and port
=====================================================

    class SIPSender
      constructor: (@socket) ->

        @socket.on 'error', (error) ->
          debug.dev "Socket error: #{error}"

        @socket.once 'listening', =>
          address = @socket.address()
          debug "Listening for messages on #{address.address}:#{address.port}"

        return

We route based on the URI domain, as per RFC.

      resolve: (dest,method,content) ->
        return unless dest?
        addresses = await resolve dest.uri

        await Promise.all addresses.map (address) =>
          @send dest, method, content, address

        return

      publish: (dest,content) ->
        @resolve dest,'PUBLISH',content

      notify: (dest,content) ->
        @resolve dest,'NOTIFY',content

      send: (dest,method,content,target) ->

        headers = Buffer.from """
          #{method} sip:#{dest.uri} SIP/2.0
          Via: SIP/2.0/UDP #{target.name}:#{target.port};branch=0
          Max-Forwards: 2
          To: <sip:#{dest.to}>
          From: <sip:#{dest.to}>;tag=#{Math.random()}
          Call-ID: five-toes-#{Math.random()}
          CSeq: 1 #{method}
          Event: #{content.event}
          Subscription-State: active
          Content-Type: #{content.type}
          Content-Length: #{content.body.length}
          X-En: #{dest.endpoint}
          \n
        """.replace /\n/g, "\r\n"

        message = Buffer.allocUnsafe headers.length + content.body.length
        headers.copy message
        content.body.copy message, headers.length

        await new Promise (resolve,reject) =>
          @socket.send message, 0, message.length, target.port, target.name, (error) ->
            if error
              reject error
            else
              resolve()
            return
        return

    module.exports = SIPSender
