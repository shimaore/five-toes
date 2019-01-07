    ({expect} = require 'chai').should()

    provisioning = "http://#{process.env.COUCHDB_USER}:#{process.env.COUCHDB_PASSWORD}@couchdb:5984/provisioning"
    before ->
      CouchDB = require 'most-couchdb'
      prov = new CouchDB provisioning
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

    describe 'Modules', ->
      it 'should load', -> require '..'

    describe 'Message Summary', ->
      Content = require '../content'
      it 'should return message summary', ->
        m = require '../message-summary'
        o = m 1,2
        o.should.be.an.instanceof Content
        o.should.have.property 'event', 'message-summary'
        o.should.have.property 'content_type'
        o.content_type.should.be.a 'string'
        o.should.have.property 'body'
        o.body.should.be.an.instanceof Buffer

    describe 'Yealink Message', ->
      Content = require '../content'
      it 'should return yealink message', ->
        m = require '../yealink-message'
        o = m 1,2
        o.should.be.an.instanceof Content
        o.should.have.property 'event', 'Yealink-xml'
        o.should.have.property 'content_type'
        o.content_type.should.be.a 'string'
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
