Main Runtime Client File

	Meteor.startup () ->
		#Deps.autorun () -> Meteor.subscribe "contacts"
		if not Meteor.userId()?
			null
			#Meteor.call "resetXing"
			#Meteor.call "initializeXing"
			
			#Meteor.call "initializeXing", (err, res) -> if res?.success isnt true then throw {
			#	msg: "Couldn't get XING-token -> this will not work, please try and reload in a few moments!"
			#}

			#Meteor.call "initializeXing", (err, res) -> Xing.requestToken = res.requestToken