<template>
    <div :id="'glassCanvas-' + canvas_no" class="glass-break-canvas"></div>
</template>

<script>
    export default {
        name: 'shardCanvas',
        props: {
            canvas_no: Number,
            canvas_id: String,
            shard_cnt: Number,
            shard_size: Number

        },
        data() {
            return {
                canvasedSketch : "",
            }
        },
        mounted() {
            /* ######### P5 stuff ######### */

            let style_rules = getComputedStyle(document.body);
            let props = this.$props;
            let glass_color = style_rules.getPropertyValue("--clear-glass-light");
            let shardSize = 35;
            let chaffSize = 3;
            let chaffMultiplier = 3.5;
            let variance = 20;
            // Max preset horizontal velocity.
            let MPH_velocity = 4;
            // Max preset horizontal velocity.
            let MPV_velocity = 10;
            let g_acceleration = 0.6;
            let maxVelocity = 9.8;
            let crack_offset = {
            x: -8,
            y: -9
            };
            let canvas_dims = {
                x: 400,
                y: 1000
            };
            let canvas_offset = 100;
            let canvasShards = [];

            // Example for creating multiple p5 canvases.
            // https://editor.p5js.org/caminofarol/sketches/r609C2cs

            p5.disableFriendlyErrors = true;

            let newSketch = function (sketch) {
                let canvas_data_id = props.canvas_id;
                let canvas_data_parent = 'glassCanvas-' + props.canvas_no;
                let canvas_data_shard_cnt = props.shard_cnt;
                let canvas_data_shard_size = props.shard_size;
                let fill_c = sketch.color(glass_color);

                sketch.setup = function () {
                    let glassCanvas = sketch.createCanvas(canvas_dims.x, canvas_dims.y);
                    glassCanvas.id(canvas_data_id);
                    glassCanvas.parent(canvas_data_parent);
                };

                sketch.draw = function () {
                    sketch.clear();
                    sketch.fill(fill_c);
                    sketch.strokeWeight(1);
                    sketch.stroke(sketch.color("rgba(255,255,255,0.8)"));
                    if (canvasShards) {
                        canvasShards.forEach(function (element, SHindex) {
                            if (element.deletable) {
                                canvasShards.splice(SHindex, 1);
                            }
                            element.update();
                            element.display(sketch);
                        });
                    }
                    else {
                        // When we dont need to draw, we should end the loop.
                        // https://p5js.org/reference/#/p5/loop
                        sketch.noLoop();
                    }
                };

                sketch.makeShards = function () {
                    canvasShards = shardsCreate(
                        canvas_data_shard_cnt,
                        canvas_data_shard_size
                    );
                };
            };

            // Set up the canvases.
            this.canvasedSketch = new p5(newSketch);

            function shardsCreate(shard_cnt, shard_size) {
            let shards = [];
                for (let i = 0; i < shard_cnt; i++) {
                    let ranposX = Math.floor(Math.random() * 50) - 25;
                    let ranposY = Math.floor(Math.random() * 50) - 50;
                    shards.push(
                    new Shard(
                        canvas_dims.x / 2 + ranposX - 10,
                        canvas_offset + ranposY,
                        shard_size
                    )
                    );
                }
                for (let i = 0; i < shard_cnt * chaffMultiplier; i++) {
                    let ranposX = Math.floor(Math.random() * 50) - 25;
                    let ranposY = Math.floor(Math.random() * 50) - 50;
                    shards.push(
                        new Shard(canvas_dims.x / 2 + ranposX - 10, canvas_offset + ranposY, null)
                    );
                }

                return shards;
            }

            // https://www.w3schools.com/js/js_object_constructors.asp
            // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects
            function Shard(iPosX, iPosY, size) {
                // Variables.
                this.pos = {
                    x: iPosX,
                    y: iPosY
                };

                this.chaff = true;
                if (size !== null && size > chaffSize) {
                    this.chaff = false;
                }

                // Maximum rotation speed.
                // this.MR_speed = 3;
                // Rotation per frame.
                // this.rpf =
                //     Math.floor(Math.random() * (this.MR_speed * 2 + 1)) - this.MR_speed;
                // Current rotation.
                // this.rotation = 0;
                this.deletable = false;

                this.shard_variance = Math.floor(Math.random() * variance) - variance / 2;

                // https://www.w3schools.com/js/js_random.asp
                this.vertices_cnt = Math.floor(Math.random() * 3) + 3;
                if (this.chaff === true) {
                    this.vertices = sortPoints(
                    randomPoints(this.vertices_cnt, chaffSize, chaffSize),
                    chaffSize,
                    chaffSize
                    );
                } else {
                    this.vertices = sortPoints(
                    randomPoints(
                        this.vertices_cnt,
                        shardSize + this.shard_variance,
                        shardSize + this.shard_variance
                    ),
                    shardSize,
                    shardSize
                    );
                }
                this.horizontalVelocity = Math.random() * MPH_velocity - MPH_velocity / 2;
                this.verticalVelocity = Math.random() * MPV_velocity - MPV_velocity / 2;

                // Functions.
                this.update = function () {
                    this.verticalVelocity += g_acceleration;
                    if (this.verticalVelocity >= maxVelocity) {
                    this.verticalVelocity = maxVelocity;
                    }
                    // this.rotation += this.rpf;

                    this.pos = {
                        x: this.pos.x + this.horizontalVelocity,
                        y: this.pos.y + this.verticalVelocity
                    };

                    if (this.pos.y > canvas_dims.y) {
                        this.deletable = true;
                    }
                };

                this.display = function (sketch) {
                    // if (this.chaff) {
                    //   return;
                    // }

                    sketch.push();

                    // DEBUG::SHOWS OFFSET CENTER.
                    // sketch.circle(
                    //   canvas_dims.x / 2 + crack_offset.x,
                    //   canvas_offset + crack_offset.y,
                    //   10
                    // );

                    // sketch.angleMode(sketch.DEGREES);
                    sketch.translate(
                        this.vertices[1][0] + this.pos.x + crack_offset.x,
                        this.vertices[1][1] + this.pos.y + crack_offset.y
                    );

                    // DEBUG::SHOWS ROTATION CENTER.
                    // circle(0, 0, 10);
                    // sketch.rotate(this.rotation);
                    sketch.translate(-this.vertices[1][0], -this.vertices[1][1]);
                    sketch.beginShape();
                    for (let i = 0; i < this.vertices[0].length; i++) {
                        sketch.vertex(this.vertices[0][i][0], this.vertices[0][i][1]);
                    }
                    sketch.endShape(sketch.CLOSE);
                    sketch.pop();
                };
            }

            // BIG props.
            // https://observablehq.com/@tarte0/generate-random-simple-polygon
            function randomPoints(n, width, height) {
            const points = [];
            for (let i = 0; i < n; i++) {
                points.push([Math.random() * width, Math.random() * height]);
            }
            return points;
            }

            function sortPoints(points, width, height) {
            const centerPoint = [width / 2, height / 2];
            const sorted = points.slice(0);
            let angleFromCenter;

            const sortByAngle = (p1, p2) => {
                return (
                (Math.atan2(p1[1] - centerPoint[1], p1[0] - centerPoint[0]) * 180) /
                    Math.PI -
                (Math.atan2(p2[1] - centerPoint[1], p2[0] - centerPoint[0]) * 180) /
                    Math.PI
                );
            };

            sorted.sort(sortByAngle);
            return [sorted, centerPoint];
            }

        },
        methods: {
            makeShards() {
                this.canvasedSketch.loop();
                this.canvasedSketch.makeShards();
            }
        }
    }
</script>

<style lang="scss">
    div[id^="glassCanvas"] {
        position: fixed;
        pointer-events: none;
        width: 100vw;
        height: 100vh;
        left: 0;
        top: 0;

        canvas {
            position: absolute;
            pointer-events: none;
            filter: drop-shadow(10px 3px 10px rgba(0, 0, 0, 0.6))
                drop-shadow(-10px -3px 10px rgba(0, 0, 0, 0.6));
            opacity: 0;
            filter: drop-shadow(10px 3px 10px rgba(0, 0, 0, 0.6))
                drop-shadow(-10px -3px 10px rgba(0, 0, 0, 0.6));

            &.cracked {
                opacity: 1;
            }
        }
    }
</style>