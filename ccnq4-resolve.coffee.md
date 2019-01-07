    module.exports = ccnq4_resolve = (cfg) ->

      prov = new CouchDB cfg.provisioning

      (local_number) ->

Collect the endpoint/via fields from the local number.

        number_doc = await get_prov prov, "number:#{local_number}"
        return if number_doc.disabled

Use the endpoint name and via to route the packet.

        endpoint = number_doc.endpoint
        via = number_doc.endpoint_via

        debug 'resolve', {via,endpoint}
        return unless endpoint?

Registered endpoint

        if m = endpoint.match /^([^@]+)@([^@]+)$/
          to = endpoint
          if via?
            uri = [m[1],via].join '@'
          else
            uri = endpoint
          return {uri,to,endpoint}

Static endpoint

        else
          if via?
            to = [local_number,endpoint].join '@'
            uri = [local_number,via].join '@'
            return {uri,to,endpoint}
          else
            debug 'No `via` for static endpoint, skipping.'

        return

    CouchDB = require 'most-couchdb'
    get_prov = require './get-prov'
    {debug} = (require 'tangible') 'five-toes:ccnq4-resolve'
