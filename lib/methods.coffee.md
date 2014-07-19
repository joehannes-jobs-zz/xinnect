This is the principal file for server methods accessible from the client
	
	if Meteor.isServer

		Meteor.methods {
			deprecateXing: () -> ServiceConfiguration.configurations.remove service: "xing"
			prepareXing: () ->
				if not ServiceConfiguration.configurations.findOne({ service: "xing" })? then ServiceConfiguration.configurations.insert
					service: "xing"
					consumerKey: '8a82af1b01b3046bb6c7'
					secret: OAuth.sealSecret "1f8df2941330d8616489a3209adc8c55a8864801"
					callbackUrl: Meteor.absoluteUrl "_oauth/xing?close"
			get: () ->
				contacts = Meteor.user()?.profile?.contacts
				if contacts?
					for contact, i in contacts when Contacts.find({ id: contact, user: Meteor.userId() }).count() is 0
						details = Xing.call("users/" + contact).contact
						random_cat = Math.round Math.random 2
						console.log "loading #{i} of #{contacts.length} ..."
						Contacts.upsert details.id, { $set: {
							id: details.id
							user: Meteor.userId()
							display_name: details.first_name
							avatar: details.photo_urls.medium_thumb
							telephone: details.business_address.phone
							category: ["Manager", "Marketing", "Techniker"][random_cat]
							category_color: ["red", "green", "blue"][random_cat]
							visibility: "visible"
						}}
					success: true
				else success: false
			addConnection: (id) -> 
				Contacts.upsert { id: id, user: Meteor.userId() },  { $set: { visibility: "hidden" } }
				if Connections.find({ user: id }).count() is 0 
					contacts = null
					query = null
					bar = () ->
						try
							query = Xing.call("users/#{id}/contacts", limit: 0, "GET", true)
						catch e
							console.log e
							bar()
					bar()
					worker = total = query.contacts.total
					foo = () ->
						try
							contacts = ({ id: contact.id, avatar: contact.photo_urls.medium_thumb } for contact in Xing.call("users/#{id}/contacts", { limit: 100, offset: total - worker, user_fields: ["id", "photo_urls"] }, "GET", true).contacts.users while worker > 0 and worker-=100).flatten()
						catch e
							console.log e
							foo()
					foo()
					Connections.insert { 
						contacts: contacts
						user: id
					}
				if Graph.find( user: Meteor.userId() ).fetch()?[0]?.contacts?.length > 0 then Graph.upsert { user: Meteor.userId() }, { $addToSet: { contacts: id } }
				else Graph.upsert { user: Meteor.userId() }, { $set: { contacts: [id] } }
			volatileGraph: (state) ->
		}