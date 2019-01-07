    ({expect} = require 'chai').should()

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


    describe 'ccnq4-resolve', ->
      m = require '../ccnq4-resolve'
      it 'should return a function', ->
        o = m provisioning:'hello'
        o.should.be.a 'function'
