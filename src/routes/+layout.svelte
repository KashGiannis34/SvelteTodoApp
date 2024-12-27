<script>
    import { auth, db } from "../lib/firebase/firebase";
    import { onMount } from "svelte";
    import { getDoc,doc,setDoc } from "firebase/firestore";
    import { authStore } from "../store/store";

    // routes that don't need authentication for access
    const nonAuthRoutes = ['/'];

    // when the site is loading
    onMount(() => {
        console.log("Mounting");
        // called when doing log in, log out, or registration
        // when logging out, user is null
        const unsubscribe = auth.onAuthStateChanged(async user => {
            const currentPath = window.location.pathname;

            // return to authentication page
            if (!user && !nonAuthRoutes.includes(currentPath)) {
                window.location.href = "/";
                return;
            }

            // log in to dashboard if a user exists
            if (user && currentPath === "/") {
                window.location.href = "/dashboard";
                return;
            }

            // don't do anything
            if (!user) {
                return;
            }

            // gather user data
            let dataToSetToStore;
            const docRef = doc(db, 'users', user.uid);
            const docSnap = await getDoc(docRef);

            // if the user doesn't exist, initialize the user's data (register basically)
            if (!docSnap.exists()) {
                const userRef = doc(db, 'users', user.uid);
                dataToSetToStore = {
                    email: user.email,
                    todos: []
                };

                await setDoc(
                    userRef,
                    {
                        email: user.email,
                        todos: []
                    },
                    { merge: true }
                );
            } else {
                // access user data
                const userData = docSnap.data();
                dataToSetToStore = userData;
            }

            // update auth data
            authStore.update((curr) => {
                return {
                    ...curr,
                    user,
                    data: dataToSetToStore,
                    loading: false // stop loading because the page has been successfully loaded
                };
            });

        })
    })
</script>

<div class="mainContainer">
    <slot/>
</div>

<style>
    .mainContainer {
        min-height: 100vh;
        background: linear-gradient(to right, #000428, #000046);
        color: white;
        position: relative;
        display: flex;
        flex-direction: column;
    }
</style>