<template>
    <div class="unit-item" :style="transformation">
        <div class="unit-item__header">
            <div class="unit-item__header__text single-line input"
            :contenteditable="editmode"
            @keydown="keydown">
                {{unit.getText()}}
            </div>  
            <div class="unit-item__header__type">I</div>
        </div>
        <div class="connector">
            <div class="connector-left"></div>
            <div class="connector-right"></div>
            <div class="connector-enum"></div>
        </div>
    </div>
</template>

<script>
/*
* Calculates the angle between AB and the X axis
* A and B are points (ax,ay) and (bx,by)
*/
// function getAngleDeg(ax,ay,bx,by) {
//     var angleRad = Math.atan((ay-by)/(ax-bx));
//     var angleDeg = angleRad * 180 / Math.PI;
    
//     return(angleDeg);
// }

import Unit from '../Unit.js'
import {Point} from '../../assets/js/unit-classes';

export default {
    mixins: [Unit],
    mounted: function() {
        /* Position element as saved in DB */
        this.left = this.unit.xpos();
        this.top = this.unit.ypos();

        /* Set edit functionality */
        this.handleEditLogic();
    },
    methods: {
        /* Drag & Drop methods  */
        onDragStart: function() {
            /* Update global z-index and future values */
            this.$el.dispatchEvent(new Event('unitdragstart'));
            
            /* Move the element upwards */
            this.$el.style.zIndex = this.zIndex;

            if (this.groupContainer) {
                this.$el.style.zIndex -= 500;
            }

            /* If we are during a relation linkage process */
            if (document.body.style.cursor === 'cell') {
                document.body.style.cursor = 'default';
                this.$parent.createRelation(null, this.unit.getUID());           
            }
        },
        onDrag: function (dx,dy) {
            /* Position element in the UI level */
            this.moveBy(dx,dy);
        },
        onDragEnd: function() {
            this.$el.dispatchEvent(new Event('unitdragend'));
        },
        onDelete: async function () {
            let board = this.$parent;
            var x,y,point, elRect = this.$el.getBoundingClientRect();


            for (const lineId of this.lines) {
                let line = board.$refs[lineId];

                x = this.left + elRect.width/2 - 5*this.scale;
                y = this.top + elRect.height/2 - 5*this.scale;
                point = new Point(x,y,this.unit.getType());

                /* Detach connected relations upon deletion */
                if (line.src.unit.getType() === "Relation" && line.src.unit.getUID() !== this.unit.getUID()) {
                    if (line.src.unit.getDestId() === this.unit.getUID()) {
                        await board.addUnitOnRuntime(point);
                        await line.src.changeDestOnRuntime(point.getUID());
                    } else {
                        await board.deleteLineOnRuntime(lineId);
                    }
                }
                else if (line.dest.unit.getType() === "Relation" && line.dest.unit.getUID() !== this.unit.getUID()) {
                    if (line.dest.unit.getSrcId() === this.unit.getUID()) {
                        await board.addUnitOnRuntime(point);
                        await line.dest.changeSrcOnRuntime(point.getUID());
                    } else {
                        await board.deleteLineOnRuntime(lineId);
                    }
                }
                else if (line.options.isInfoToEnum) {
                    let enumUnit = line.dest;
                    enumUnit.unit.setItemId(null);
                }

                if (line.src.unit.getUID() === this.unit.getUID()) {
                    await board.deleteLineOnRuntime(lineId);
                }
                if (line.dest.unit.getUID() === this.unit.getUID()) {
                    await board.deleteLineOnRuntime(lineId);
                }
            }

            if (this.groupContainer) {
                this.groupContainer.removeItem(this.unit.getUID())
            }

            board.deleteUnitOnRuntime(this.$el.ref);                
        },
        onDuplicate() {

        },
        onEdit() {

        },
    },

}
</script>

<style lang="scss" scoped>
    .unit-item {
        display: flex;
        flex-direction: column;
        border-color: rgb(66, 178, 253); 

        &__header {
            display: flex;
            justify-content: space-between;
            font-weight: 900;

            &__text {
                order: 1;
                align-self: center;  
                margin-left: 10px;  
            }
            &__type {
                order: 0;
                border-radius: 100%;
                color: black;
                text-align: center;
                height: 30px;
                width: 30px;
                margin: auto;
                margin-left: 0;
                font-size: 17px;
                outline:solid black 1px;
                align-self: center;   
            }
        }

        .connector {
            // position: absolute;
            &-enum {
                position: absolute;
                top: 50%;
                left: 50%;
            }
            &-left {
                position: absolute;
                top: 50%;
                left: 0%;
            }
            &-right {
                position: absolute;
                top: 50%;
                left: 100%;
            }
        }
    }

</style>