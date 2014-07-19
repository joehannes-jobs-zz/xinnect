Xing Client Side Meteor Accounts Integration

	if Meteor.isClient

		class @Xing
			@requestCredential: (options, credentialRequestCompleteCallback) ->
				if not credentialRequestCompleteCallback and Object.isFunction options
					credentialRequestCompleteCallback = options
					options = null

				credentialToken = Random.secret()
				loginUrl = '/_oauth/xing/?requestTokenAndRedirect=true&state=' + credentialToken

				#OAuth.showPopup loginUrl, _.bind(credentialRequestCompleteCallback, null, credentialToken)

				#loginUrl = "https://api.xing.com/v1/authorize?oauth_token=" + Xing.requestToken.token
				Oauth.initiateLogin credentialToken, loginUrl, credentialRequestCompleteCallback, { width: 900, height: 450 }