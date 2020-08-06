<template>
    <div>
        <span
            v-for="key in Object.keys(otherCursors)"
            style="position: absolute"
            :key="key"
            :ref="key"
        >🖱️</span>
    </div>
</template>

<script>
import { HubConnectionBuilder } from "@microsoft/signalr";

export default {
    data: () => ({
        cursor: { x: 0, y: 0 },
        otherCursors: {},
        connections: null,
    }),
    methods: {
        getCursor(e) {
            const vm = this;
            vm.cursor.x = window.Event
                ? e.pageX
                : event.clientX +
                  (document.documentElement.scrollLeft
                      ? document.documentElement.scrollLeft
                      : document.body.scrollLeft);
            vm.cursor.y = window.Event
                ? e.pageY
                : event.clientY +
                  (document.documentElement.scrollTop
                      ? document.documentElement.scrollTop
                      : document.body.scrollTop);
        },
    },
    watch: {
        cursor: {
            handler() {
                const vm = this;
                vm.connection.invoke("SendCursor", vm.cursor);
            },
            deep: true,
        },
    },
    async mounted() {
        const vm = this;
        vm.connection = new HubConnectionBuilder()
            .withUrl("https://localhost:5001/CursorHub")
            .build();

        vm.connection.on("receiveCursor", (userId, cursor) => {
            if (vm.$refs[userId]) {
                vm.$refs[userId][0].style.left = `${cursor.x}px`;
                vm.$refs[userId][0].style.top = `${cursor.y}px`;
            }
            vm.$set(vm.otherCursors, userId, cursor);
        });

        await vm.connection.start();

        if (window.Event) {
            document.captureEvents(Event.MOUSEMOVE);
        }
        document.onmousemove = vm.getCursor;
    },
};
</script>