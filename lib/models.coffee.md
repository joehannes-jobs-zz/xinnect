Models <=> Collections

	@Dummys = new Meteor.Collection "dummys"
	@Contacts = new Meteor.Collection "contacts"
	@Comments = new Meteor.Collection "comments"
	@Journalists = new Meteor.Collection "journalists"
	@Connections = new Meteor.Collection "connections"
	@Graph = new Meteor.Collection "graph"
	@PredefinedUsers = new Meteor.Collection "predefinedusers"

	if Meteor.isServer

		dummys = [{
			user: 0
			display_name: "Joehannes"
			avatar: "joehannes.jpg"
			telephone: "+43-660-4189546"
			category_color: "red"
		}, {
			user: 0
			display_name: "Warrior"
			avatar: "ateam.jpg"
			telephone: "+43-660-4189546"
			category_color: "blue"
		}, {
			user: 0
			display_name: "Ninja"
			avatar: "ninja.jpg"
			telephone: "+43-660-4189546"
			category_color: "green"
		}]

		comments = [{
			name: "Joehannes"
			avatar: "joehannes.jpg"
			comment: "❤ Welcome to the bright side of life ❤"
		}]

		journalists = [{
			name: "Joehannes"
			avatar: "joehannes.jpg"
		}, {
			name: "Markus"
			avatar: "markus.jpg"
		}, {
			name: "Guenther"
			avatar: "guenther.jpg"
		}]

		predefinedusers = [{
			username: 'johannes'
			email: 'johannes.neugschwentner@gmail.com'
			password: 'M4ri4Luis4'
		}, {
			username: 'markus'
			email: 'markus.neuwirth@gmail.com'
			password: 'wisdomteeth'
		}, {
			username: 'guenther'
			email: 'guenther@klement.cc'
			password: 'smoothgesture'
		}]		

		Meteor.startup () -> 
			Dummys.remove {}
			Comments.remove {}
			Journalists.remove {}
			PredefinedUsers.remove {}
			Graph.remove {}

			Dummys.insert(d) for d in dummys
			Comments.insert(d) for d in comments
			Journalists.insert(d) for d in journalists
			PredefinedUsers.insert(d) for d in predefinedusers

			Meteor.publish "dummys", (query = {}) -> Dummys.find query
			Meteor.publish "contacts", (query = {}) -> Contacts.find query
			Meteor.publish "comments", (query = {}) -> Comments.find query
			Meteor.publish "journalists", (query = {}) -> Journalists.find query
			Meteor.publish "connections", (query = {}) -> Connections.find query
			Meteor.publish "graph", (query = {}) -> Graph.find query