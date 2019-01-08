    ({expect} = require 'chai').should()
    dgram = require 'dgram'

    port = 7124

    provisioning = "http://#{process.env.COUCHDB_USER}:#{process.env.COUCHDB_PASSWORD}@couchdb:5984/provisioning"
    before ->
      CouchDB = require 'most-couchdb'
      prov = new CouchDB provisioning
      await prov.destroy().catch -> yes
      await prov.create()
      await prov.put
        _id: 'number:lily@example.com'
        endpoint: 'phone1@proxy1.example.com'
      await prov.put
        _id: 'number:lila@example.com'
        endpoint: 'phone2@example.com'
        endpoint_via: 'proxy2.example.com'
      await prov.put
        _id: 'number:lolo@example.com'
        endpoint: '192.168.1.2'
        endpoint_via: 'proxy3.example.com'
      await prov.put
        _id: 'endpoint:anna@example.com'
        number_domain: 'phone.example.com'

    describe 'Modules', ->
      it 'should load', -> require '..'

    describe 'Message Summary', ->
      Content = require '../content'
      it 'should return message summary', ->
        m = require '../message-summary'
        o = m 1,2
        o.should.be.an.instanceof Content
        o.should.have.property 'event', 'message-summary'
        o.should.have.property 'type'
        o.type.should.be.a 'string'
        o.should.have.property 'body'
        o.body.should.be.an.instanceof Buffer

    describe 'Yealink Message', ->
      Content = require '../content'
      it 'should return yealink message', ->
        m = require '../yealink-message'
        o = m 1,2
        o.should.be.an.instanceof Content
        o.should.have.property 'event', 'Yealink-xml'
        o.should.have.property 'type'
        o.type.should.be.a 'string'
        o.should.have.property 'body'
        o.body.should.be.an.instanceof Buffer

    describe 'resolve', ->
      m = require '../resolve'

      it 'should resolve with domain', ->
        o = await m 'anonymous@a.phone.kwaoo.net'
        o.should.be.an('array').of.length 1
        o.should.have.property 0
        o[0].should.have.property 'name'
        o[0].should.have.property 'port', 5060

      it 'should resolve with port', ->
        o = await m 'anonymous@a.phone.kwaoo.net:5070'
        o.should.be.an('array').of.length 1
        o.should.have.property 0
        o[0].should.have.property 'name'
        o[0].should.have.property 'port', 5070

      it 'should cache', ->
        o = await m 'anonymous@a.phone.kwaoo.net:5070'
        o = await m 'anonymous@a.phone.kwaoo.net:5070'
        o.should.be.an('array').of.length 1
        o.should.have.property 0
        o[0].should.have.property 'name'
        o[0].should.have.property 'port', 5070

      it 'should fail', ->
        o = try await m 'anonymous@example.net'
        expect(o).to.be.undefined

    describe 'ccnq4-resolve', ->
      m = require '../ccnq4-resolve'
      it 'should return a function', ->
        o = m {provisioning}
        o.should.be.a 'function'

      it 'should evaluate the function', ->
        f = m {provisioning}
        o = await f 'bob@example.net'
        expect(o).to.be.undefined

      it 'should resolve endpoints', ->
        f = m {provisioning}
        o = await f 'lily@example.com'
        expect(o).to.be.an 'object'
        o.should.have.property 'uri', 'phone1@proxy1.example.com'
        o.should.have.property 'endpoint', 'phone1@proxy1.example.com'
        o.should.have.property 'to', 'phone1@proxy1.example.com'

      it 'should resolve endpoints', ->
        f = m {provisioning}
        o = await f 'lila@example.com'
        expect(o).to.be.an 'object'
        o.should.have.property 'uri', 'phone2@proxy2.example.com'
        o.should.have.property 'endpoint', 'phone2@example.com'
        o.should.have.property 'to', 'phone2@example.com'

      it 'should resolve endpoints', ->
        f = m {provisioning}
        o = await f 'lolo@example.com'
        expect(o).to.be.an 'object'
        o.should.have.property 'uri', 'lolo@proxy3.example.com'
        o.should.have.property 'endpoint', '192.168.1.2'
        o.should.have.property 'to', 'lolo@192.168.1.2'

    describe 'ccnq4-receiver', ->
      m = require '../ccnq4-receiver'
      it 'should return a function', ->
        o = m {provisioning}
        o.should.be.a 'function'

      it 'should evaluate the function', (done) ->
        socket = dgram.createSocket 'udp4'
        after -> socket.close()

        f = m {provisioning}
        f socket, ({number_domain,user_id}) ->
          number_domain.should.eql 'phone.example.com'
          user_id.should.eql 'anna@phone.example.com'
          done()

        msg = Buffer.from '''
          SUBSCRIBE sip:anna@example.com SIP/2.0
          Via: SIP/2.0/UDP 127.0.0.1:5060;branch=0
          Call-ID: 123
          CSeq: 1 SUBSCRIBE
          Event: message-summary
          From: <sip:bob@example.com>;tag=43
          To: <sip:anna@example.com>
          X-En: anna@example.com
          Content-Length: 0
          \n
        '''.replace /\n/g, '\r\n'
        socket.bind port
        socket.send msg, 0, msg.length, port, '127.0.0.1'
        port++
        return

    describe 'sip-sender', ->
      m = require '../sip-sender'
      ms = require '../message-summary'
      it 'should return a class', ->
        m.should.be.a 'function'
        socket = dgram.createSocket 'udp4'
        after -> socket.close()
        o = new m socket

      it 'should notify', (done) ->
        socket = dgram.createSocket 'udp4'
        after -> socket.close()
        o = new m socket

        socket.on 'message', (msg) ->
          console.log msg.toString()
          msg.toString().should.match /^NOTIFY /
          done()

        socket.bind port

        dest =
          to: "bob@localhost:#{port}"
          uri: "bob@localhost:#{port}"
          endpoint: 'bob'

        content = ms 2,3


        o.notify dest, content
        port++
        return

      it 'should publish', (done) ->
        socket = dgram.createSocket 'udp4'
        after -> socket.close()
        o = new m socket

        socket.on 'message', (msg) ->
          console.log msg.toString()
          msg.toString().should.match /^PUBLISH /
          done()

        socket.bind port

        dest =
          to: "bob@localhost:#{port}"
          uri: "bob@localhost:#{port}"
          endpoint: 'bob'

        content = ms 2,3


        o.publish dest, content
        port++
        return
