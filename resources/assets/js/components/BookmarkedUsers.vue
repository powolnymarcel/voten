<template>
	<section>
        <bookmarked-user v-for="user in users" :list="user" :key="user.id"></bookmarked-user>

	    <no-content v-if="nothingFound" :text="'You have not bookmarked any users yet'"></no-content>

		<loading v-show="loading"></loading>

		<no-more-items :text="'No more items to load'" v-if="NoMoreItems && !nothingFound"></no-more-items>
	</section>
</template>

<script>
    import Loading from '../components/Loading.vue'
    import BookmarkedUser from '../components/BookmarkedUser.vue'
	import NoMoreItems from '../components/NoMoreItems.vue'
    import NoContent from '../components/NoContent.vue'

    export default {
        components: {
        	NoContent,
        	BookmarkedUser,
        	Loading,
			NoMoreItems
        },

        data: function () {
            return {
	            NoMoreItems: false,
				loading: true,
                nothingFound: false,
                users: [],
				page: 0
            }
        },

        created () {
			this.$eventHub.$on('scrolled-to-bottom', this.loadMore)
            this.getUsers()
        },

	    watch: {
			'$route': function () {
				this.clearContent()
				this.getUsers()
			}
		},


        methods: {
			loadMore () {
				if ( Store.contentRouter == 'content' && !this.loading && !this.NoMoreItems ) {
					this.getUsers()
				}
			},

        	clearContent () {
				this.nothingFound = false
				this.users = []
				this.loading = true
        	},

            getUsers () {
				this.loading = true
				this.page ++

            	axios.post('/bookmarked-users', {
            		page: this.page
            	}).then((response) => {
					this.users = [...this.users, ...response.data.data]

					if (response.data.next_page_url == null) {
						this.NoMoreItems = true
					}

					if(this.users.length == 0){
						this.nothingFound = true
					}

					this.loading = false
                })
            }
        }
    };
</script>
