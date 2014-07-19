Xing Server Side -- Integration with Meteor Accounts + OAuth Functionality

	if Meteor.isServer

		userAgent = "Meteor"
		if Meteor.release then userAgent += "/" + Meteor.release

		urls =
			requestToken: "https://api.xing.com/v1/request_token"
			authorize: "https://api.xing.com/v1/authorize"
			accessToken: "https://api.xing.com/v1/access_token"
			authenticate: "https://api.xing.com/v1/authorize"

		OAuth.registerService 'xing', 1, urls, (query) ->
			identity = query.get("https://api.xing.com/v1/users/me").data.users[0]
			contacts = query.get("https://api.xing.com/v1/users/me/contact_ids").data.contact_ids.items

			serviceData:
				id: identity.id,
				accessToken: OAuth.sealSecret query.accessToken
				accessTokenSecret: OAuth.sealSecret query.accessTokenSecret
				email: identity.active_email
				username: identity.active_email
			options:
				profile: 
					name: identity.first_name
					avatar: identity.photo_urls.medium_thumb
					contacts: contacts


**API like Server Methods for actually calling the Xing-API**

		Xing = class @Xing
			@retrieveCredential = (credentialToken, credentialSecret) -> OAuth.retrieveCredential credentialToken, credentialSecret
			@call: (url, params = null, method = "GET", raw = false) ->
				url = "https://api.xing.com/v1/" + url
				config = ServiceConfiguration.configurations.findOne({ service: "xing" })
				oauth = new OAuth1Binding config
				user = Meteor.user()
				oauth.accessToken = user.services.xing.accessToken
				oauth.accessTokenSecret = user.services.xing.accessTokenSecret

				if raw is true then	oauth.call(method, url, params)?.data
				else contact: oauth.call(method, url, params)?.data?.users?[0]