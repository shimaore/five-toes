    module.exports = ccnq4_receiver = (cfg) ->
      prov = new CouchDB cfg.provisioning
      ( socket, accept, handler ) ->
        if typeof accept is 'function'
          [handler,accept] = [accept,null]

        accept ?= ['SUBSCRIBE']
        ua = {}

        socket.on 'message', (msg,rinfo) =>
          debug "Received #{msg.length} bytes message from #{rinfo.address}:#{rinfo.port}"

          content = msg.toString 'ascii'
          trace 'Received message', content

The parser returns an IncomingRequest for a SUBSCRIBE message.

          request = Parser.parseMessage content, ua
          return unless request? and request.method in accept and request.event?.event is 'message-summary'

          trace request.method, {number_domain,user_id}

Try to recover the number and the endpoint from the message.

          number = request.ruri?.user ? request.from?.uri?.user
          endpoint = request.headers['X-En']?[0]?.raw

          trace 'SUBSCRIBE', {number, endpoint}

          return unless number? and endpoint?

Recover the number-domain from the endpoint.

          {number_domain} = await get_prov prov, "endpoint:#{endpoint}"
          return unless number_domain?

          user_id = "#{number}@#{number_domain}"

          trace 'SUBSCRIBE', {number_domain,user_id}
          await handler {number_domain,user_id}
          return
        return

    CouchDB = require 'most-couchdb'
    Parser = require 'jssip/lib/Parser'
    {debug} = (require 'tangible') 'five-toes:ccnq4-receiver'
    trace = debug
    get_prov = require './get-prov'
