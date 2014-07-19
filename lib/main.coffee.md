Password Protection Configuration

	Meteor.startup () ->

		if Meteor.isServer

			Meteor.startup () ->
				ServiceConfiguration.configurations.remove service: "xing"


			if Meteor.users.find({}).count() is 0
			
				Accounts.createUser {
					username: options.username
					email: options.email
					password: options.password
				} for options in PredefinedUsers.find({}).fetch()

		AccountsTemplates.configure {
			displayFormLabels: true
			continuousValidation: true
			homeRoutePath: '/home'
			signUpRoutePath: '/disallowed'
			postSignUpRoutePath: '/home'
		}

		AccountsTemplates.init()