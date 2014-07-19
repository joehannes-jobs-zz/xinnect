Globally registering the Xing Service in the Meteor Accounts System

	if Meteor.isClient

		Meteor.loginWithXing = (options, callback) ->

Support a callback without options

			if not callback? and Object.isFunction options
				callback = options
				options = null
				
			credentialRequestCompleteCallback = Accounts.oauth.credentialRequestCompleteHandler callback
			Xing.requestCredential options, credentialRequestCompleteCallback

	if Meteor.isServer
		Accounts.addAutopublishFields {
			forLoggedInUser: ['services.xing']
			forOtherUsers: []
		}