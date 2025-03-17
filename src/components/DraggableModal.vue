<template>
  <div>
    <!-- Modal buttons -->
    <div v-for="(_, name, index) in getContentSlots()" :key="name" class="inline-block">
      <!-- Custom button slot if provided -->
      <slot 
        :name="`button-${name}`" 
        :open="() => openModal(name)" 
        :is-open="isModalOpen(name)"
        :slot-name="formatSlotName(name)"
      >
        <!-- Image button if provided -->
        <div v-if="hasButtonImage(name)" @click="openModal(name)" :class="getButtonWrapperClass(name)">
          <slot :name="`button-image-${name}`"></slot>
        </div>
        
        <!-- Default button with optional custom classes -->
        <button
          v-else
          @click="openModal(name)"
          :disabled="isModalOpen(name)"
          :class="[
            getButtonClass(name),
            isModalOpen(name) 
              ? 'cursor-not-allowed opacity-60' 
              : ''
          ]"
        >
          <slot :name="`button-icon-${name}`">
            <!-- Default icon slot if provided -->
          </slot>
          <span>{{ getButtonText(name) || `Open ${formatSlotName(name)}` }}</span>
        </button>
      </slot>
    </div>

    <!-- Modal container -->
    <Teleport to="body">
      <div v-if="activeModals.length > 0" class="fixed inset-0 pointer-events-none">
        <!-- Active modals -->
        <transition-group name="modal">
          <div
            v-for="modal in visibleModals"
            :key="modal.id"
            :class="[
              'absolute shadow-lg rounded-lg overflow-hidden bg-white border border-gray-200',
              'flex flex-col pointer-events-auto'
            ]"
            :style="{
              top: `${modal.position.y}px`,
              left: `${modal.position.x}px`,
              width: `${modal.size.width}px`,
              height: `${modal.size.height}px`,
              transition: modal.isAnimating ? 'all 0.2s ease-in-out' : 'none',
              zIndex: modal.isActive ? 1000 : 100
            }"
            @mousedown="activateModal(modal.id)"
          >
            <!-- Modal header -->
            <div
              class="bg-gray-100 cursor-move flex items-center justify-between select-none"
              @mousedown.stop="startDrag($event, modal.id, 'move')"
            >
              <h3 class="font-medium text-gray-700 py-2 px-4">{{ formatSlotName(modal.name) }}</h3>
              <div class="flex space-x-2">
                <button
                  @click.stop="minimizeModal(modal.id)"
                  class="text-gray-500 hover:text-gray-700 p-2 rounded hover:bg-slate-300 focus:outline-none"
                >
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M5 10a1 1 0 011-1h8a1 1 0 110 2H6a1 1 0 01-1-1z" clip-rule="evenodd" />
                  </svg>
                </button>
                <button
                  v-if="!modal.isMaximized"
                  @click.stop="maximizeModal(modal.id)"
                  class="text-gray-500 hover:text-gray-700 p-2 rounded hover:bg-slate-300 focus:outline-none"
                >
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M3 4a1 1 0 011-1h12a1 1 0 011 1v12a1 1 0 01-1 1H4a1 1 0 01-1-1V4zm1 0v12h12V4H4z" clip-rule="evenodd" />
                  </svg>
                </button>
                <button
                  v-else
                  @click.stop="restoreModal(modal.id)"
                  class="text-gray-500 hover:text-gray-700 p-2 rounded hover:bg-gray-300 focus:outline-none"
                >
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M5 4a1 1 0 00-1 1v10a1 1 0 001 1h10a1 1 0 001-1V5a1 1 0 00-1-1H5zm1 2h8v8H6V6z" clip-rule="evenodd" />
                  </svg>
                </button>
                <button
                  @click.stop="closeModal(modal.id)"
                  class="text-gray-500 hover:text-red-500 p-2 rounded hover:bg-red-300 focus:outline-none"
                >
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd" />
                  </svg>
                </button>
              </div>
            </div>

            <!-- Modal content -->
            <div class="flex-grow overflow-auto w-full h-full">
              <div class="w-full h-full">
                <slot :name="modal.name"></slot>
              </div>
            </div>

            <!-- Resize handles -->
            <!-- Top edge -->
            <div
              v-if="!modal.isMaximized"
              class="absolute top-0 left-0 right-0 h-1 cursor-ns-resize"
              @mousedown.stop="startDrag($event, modal.id, 'resize-n')"
            ></div>
            
            <!-- Right edge -->
            <div
              v-if="!modal.isMaximized"
              class="absolute top-0 right-0 bottom-0 w-1 cursor-ew-resize"
              @mousedown.stop="startDrag($event, modal.id, 'resize-e')"
            ></div>
            
            <!-- Bottom edge -->
            <div
              v-if="!modal.isMaximized"
              class="absolute bottom-0 left-0 right-0 h-1 cursor-ns-resize"
              @mousedown.stop="startDrag($event, modal.id, 'resize-s')"
            ></div>
            
            <!-- Left edge -->
            <div
              v-if="!modal.isMaximized"
              class="absolute top-0 left-0 bottom-0 w-1 cursor-ew-resize"
              @mousedown.stop="startDrag($event, modal.id, 'resize-w')"
            ></div>
            
            <!-- Top-left corner -->
            <div
              v-if="!modal.isMaximized"
              class="absolute top-0 left-0 w-4 h-4 cursor-nwse-resize"
              @mousedown.stop="startDrag($event, modal.id, 'resize-nw')"
            ></div>
            
            <!-- Top-right corner -->
            <div
              v-if="!modal.isMaximized"
              class="absolute top-0 right-0 w-4 h-4 cursor-nesw-resize"
              @mousedown.stop="startDrag($event, modal.id, 'resize-ne')"
            ></div>
            
            <!-- Bottom-right corner -->
            <div
              v-if="!modal.isMaximized"
              class="absolute bottom-0 right-0 w-4 h-4 cursor-nwse-resize"
              @mousedown.stop="startDrag($event, modal.id, 'resize-se')"
            >
              
            </div>
            
            <!-- Bottom-left corner -->
            <div
              v-if="!modal.isMaximized"
              class="absolute bottom-0 left-0 w-4 h-4 cursor-nesw-resize"
              @mousedown.stop="startDrag($event, modal.id, 'resize-sw')"
            ></div>
          </div>
        </transition-group>

        <!-- Tab bar for minimized modals -->
        <transition name="slide-up">
          <div
            v-if="minimizedModals.length > 0"
            class="fixed bottom-0 left-0 right-0 bg-gray-100 border-t border-gray-200 px-4 py-2 flex flex-wrap gap-2 pointer-events-auto"
          >
            <button
              v-for="modal in minimizedModals"
              :key="modal.id"
              @click="restoreMinimizedModal(modal.id)"
              class="px-3 py-1 bg-white rounded shadow hover:bg-gray-50 text-sm font-medium text-gray-700 flex items-center space-x-1"
            >
              <span>{{ formatSlotName(modal.name) }}</span>
              <svg
                @click.stop="closeModal(modal.id)"
                class="h-3 w-3 text-gray-500 hover:text-red-500"
                viewBox="0 0 20 20"
                fill="currentColor"
              >
                <path
                  fill-rule="evenodd"
                  d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
                  clip-rule="evenodd"
                />
              </svg>
            </button>
          </div>
        </transition>
      </div>
    </Teleport>
  </div>
</template>

<script>
export default {
  name: 'DraggableModal',
  props: {
    buttonClasses: {
      type: Object,
      default: () => ({})
    },
    buttonTexts: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      activeModals: [],
      isDragging: false,
      dragType: null,
      currentModalId: null,
      startX: 0,
      startY: 0,
      startWidth: 0,
      startHeight: 0,
      startTop: 0,
      startLeft: 0,
      modalCounter: 0,
      savedPositions: {},
      savedSizes: {},
      windowWidth: window.innerWidth,
      windowHeight: window.innerHeight,
      defaultButtonClass: 'font-medium py-2 px-4 rounded transition-colors m-2 bg-blue-500 hover:bg-blue-600 text-white flex items-center gap-2'
    }
  },
  computed: {
    minimizedModals() {
      return this.activeModals.filter(modal => modal.isMinimized)
    },
    visibleModals() {
      // Sort modals so active ones are rendered last (on top)
      return [...this.activeModals]
        .filter(modal => !modal.isMinimized)
        .sort((a, b) => {
          if (a.isActive && !b.isActive) return 1;
          if (!a.isActive && b.isActive) return -1;
          return 0;
        });
    }
  },
  mounted() {
    window.addEventListener('mousemove', this.onMouseMove)
    window.addEventListener('mouseup', this.onMouseUp)
    window.addEventListener('resize', this.onWindowResize)
    this.onWindowResize()
  },
  beforeUnmount() {
    window.removeEventListener('mousemove', this.onMouseMove)
    window.removeEventListener('mouseup', this.onMouseUp)
    window.removeEventListener('resize', this.onWindowResize)
  },
  methods: {
    getContentSlots() {
      // Filter slots to only get content slots (those that start with 'content-')
      return Object.keys(this.$slots)
        .filter(name => name.startsWith('content-'))
        .reduce((obj, key) => {
          obj[key] = this.$slots[key]
          return obj
        }, {})
    },
    hasButtonImage(name) {
      // Check if there's a button-image slot for this content
      return !!this.$slots[`button-image-${name}`]
    },
    getButtonClass(name) {
      // Get custom button class from props or use default
      return this.buttonClasses[name] || this.defaultButtonClass
    },
    getButtonWrapperClass(name) {
      // Get custom wrapper class for image buttons
      return this.buttonClasses[`image-${name}`] || 'cursor-pointer inline-block m-2'
    },
    getButtonText(name) {
      // Get custom button text from props
      return this.buttonTexts[name]
    },
    formatSlotName(name) {
      // Convert slot name like "content-1" to "Content 1"
      return name
        .split('-')
        .map(part => part.charAt(0).toUpperCase() + part.slice(1))
        .join(' ')
    },
    isModalOpen(name) {
      return this.activeModals.some(modal => modal.name === name)
    },
    openModal(name) {
      // Check if modal is already open
      if (this.isModalOpen(name)) {
        // Find and activate the modal if it exists
        const existingModal = this.activeModals.find(modal => modal.name === name)
        if (existingModal) {
          if (existingModal.isMinimized) {
            this.restoreMinimizedModal(existingModal.id)
          } else {
            this.activateModal(existingModal.id)
          }
        }
        return
      }

      const id = `modal-${this.modalCounter++}`
      
      // Calculate responsive initial size
      const width = Math.min(this.windowWidth * 0.8, 500)
      const height = Math.min(this.windowHeight * 0.7, 400)
      
      // Calculate centered position
      const x = Math.max(0, (this.windowWidth - width) / 2)
      const y = Math.max(0, (this.windowHeight - height) / 2)
      
      const modal = {
        id,
        name,
        isMinimized: false,
        isMaximized: false,
        isActive: true,
        isAnimating: false,
        position: { x, y },
        size: { width, height }
      };

      // Set all other modals as inactive
      this.activeModals.forEach(m => {
        m.isActive = false
      })

      this.activeModals.push(modal)
    },
    closeModal(id) {
      const index = this.activeModals.findIndex(modal => modal.id === id)
      if (index !== -1) {
        this.activeModals.splice(index, 1)
        
        // If there are remaining modals, activate the last one
        if (this.activeModals.length > 0) {
          const visibleModals = this.activeModals.filter(m => !m.isMinimized)
          if (visibleModals.length > 0) {
            const lastModal = visibleModals[visibleModals.length - 1];
            this.activateModal(lastModal.id)
          }
        }
      }
    },
    minimizeModal(id) {
      const modal = this.activeModals.find(modal => modal.id === id)
      if (modal) {
        modal.isAnimating = true
        // Faster minimize animation (reduced from 300ms to 150ms)
        setTimeout(() => {
          modal.isMinimized = true
          modal.isAnimating = false
          
          // Activate the next visible modal if available
          const visibleModals = this.activeModals.filter(m => !m.isMinimized)
          if (visibleModals.length > 0) {
            this.activateModal(visibleModals[visibleModals.length - 1].id)
          }
        }, 150)
      }
    },
    maximizeModal(id) {
      const modal = this.activeModals.find(modal => modal.id === id)
      if (modal) {
        // Save current position and size for restoration
        this.savedPositions[id] = { ...modal.position }
        this.savedSizes[id] = { ...modal.size }
        
        // Animate maximizing
        modal.isAnimating = true
        
        // Maximize the modal
        modal.isMaximized = true
        modal.position = { x: 0, y: 0 }
        modal.size = {
          width: this.windowWidth,
          height: this.windowHeight
        }
        
        // End animation after transition (faster animation)
        setTimeout(() => {
          modal.isAnimating = false
        }, 200)
      }
    },
    restoreModal(id) {
      const modal = this.activeModals.find(modal => modal.id === id)
      if (modal && modal.isMaximized) {
        modal.isAnimating = true
        modal.isMaximized = false
        
        // Restore saved position and size
        if (this.savedPositions[id]) {
          modal.position = { ...this.savedPositions[id] }
        }
        if (this.savedSizes[id]) {
          modal.size = { ...this.savedSizes[id] }
        }
        
        // Ensure the modal is within screen bounds
        this.ensureModalInBounds(modal)
        
        // End animation after transition (faster animation)
        setTimeout(() => {
          modal.isAnimating = false
        }, 200)
      }
    },

    bringToFront(modalId) {
        modal.style.zIndex = zIndexCounter++;
    },

    restoreMinimizedModal(id) {
      const modal = this.activeModals.find(modal => modal.id === id)
      if (modal) {
        modal.isMinimized = false
        this.activateModal(id)
      }
    },
    activateModal(id) {
      console.log('Activating modal:', id);
      
      // Set all modals as inactive
      this.activeModals.forEach(modal => {
        modal.isActive = modal.id === id;
      });
      
      // Force a re-render to ensure z-index changes are applied
      this.$forceUpdate();
      
    },
    startDrag(event, id, type) {
      // Prevent default to avoid text selection during drag
      event.preventDefault()
      
      const modal = this.activeModals.find(modal => modal.id === id)
      if (modal && !modal.isMaximized) {
        this.isDragging = true
        this.dragType = type
        this.currentModalId = id
        this.startX = event.clientX
        this.startY = event.clientY
        this.startWidth = modal.size.width
        this.startHeight = modal.size.height
        this.startTop = modal.position.y
        this.startLeft = modal.position.x
        this.activateModal(id)
      }
    },
    onMouseMove(event) {
      if (!this.isDragging || !this.currentModalId) return
      
      const modal = this.activeModals.find(modal => modal.id === this.currentModalId)
      if (!modal) return
      
      const deltaX = event.clientX - this.startX
      const deltaY = event.clientY - this.startY
      
      // Handle different drag types
      switch (this.dragType) {
        case 'move':
          // Move the modal
          let newX = this.startLeft + deltaX
          let newY = this.startTop + deltaY
          
          // Keep modal within window bounds
          newX = Math.max(0, Math.min(newX, this.windowWidth - modal.size.width))
          newY = Math.max(0, Math.min(newY, this.windowHeight - modal.size.height))
          
          modal.position.x = newX
          modal.position.y = newY
          break
          
        case 'resize-e':
          // Resize from right edge
          let newWidth = this.startWidth + deltaX
          newWidth = Math.max(200, Math.min(newWidth, this.windowWidth - modal.position.x))
          modal.size.width = newWidth
          break
          
        case 'resize-s':
          // Resize from bottom edge
          let newHeight = this.startHeight + deltaY
          newHeight = Math.max(150, Math.min(newHeight, this.windowHeight - modal.position.y))
          modal.size.height = newHeight
          break
          
        case 'resize-w':
          // Resize from left edge
          let widthChange = -deltaX
          let newWidthW = this.startWidth + widthChange
          
          if (newWidthW >= 200) {
            let newLeftW = this.startLeft - widthChange
            newLeftW = Math.max(0, newLeftW)
            
            // Adjust width if we hit the left boundary
            if (newLeftW === 0) {
              newWidthW = this.startWidth + this.startLeft
            }
            
            modal.size.width = newWidthW
            modal.position.x = newLeftW
          }
          break
          
        case 'resize-n':
          // Resize from top edge
          let heightChange = -deltaY
          let newHeightN = this.startHeight + heightChange
          
          if (newHeightN >= 150) {
            let newTopN = this.startTop - heightChange
            newTopN = Math.max(0, newTopN)
            
            // Adjust height if we hit the top boundary
            if (newTopN === 0) {
              newHeightN = this.startHeight + this.startTop
            }
            
            modal.size.height = newHeightN
            modal.position.y = newTopN
          }
          break
          
        case 'resize-se':
          // Resize from bottom-right corner
          let newWidthSE = this.startWidth + deltaX
          let newHeightSE = this.startHeight + deltaY
          
          newWidthSE = Math.max(200, Math.min(newWidthSE, this.windowWidth - modal.position.x))
          newHeightSE = Math.max(150, Math.min(newHeightSE, this.windowHeight - modal.position.y))
          
          modal.size.width = newWidthSE
          modal.size.height = newHeightSE
          break
          
        case 'resize-sw':
          // Resize from bottom-left corner
          let widthChangeSW = -deltaX
          let newWidthSW = this.startWidth + widthChangeSW
          let newHeightSW = this.startHeight + deltaY
          
          newHeightSW = Math.max(150, Math.min(newHeightSW, this.windowHeight - modal.position.y))
          
          if (newWidthSW >= 200) {
            let newLeftSW = this.startLeft - widthChangeSW
            newLeftSW = Math.max(0, newLeftSW)
            
            // Adjust width if we hit the left boundary
            if (newLeftSW === 0) {
              newWidthSW = this.startWidth + this.startLeft
            }
            
            modal.size.width = newWidthSW
            modal.position.x = newLeftSW
          }
          
          modal.size.height = newHeightSW
          break
          
        case 'resize-ne':
          // Resize from top-right corner
          let newWidthNE = this.startWidth + deltaX
          let heightChangeNE = -deltaY
          let newHeightNE = this.startHeight + heightChangeNE
          
          newWidthNE = Math.max(200, Math.min(newWidthNE, this.windowWidth - modal.position.x))
          
          if (newHeightNE >= 150) {
            let newTopNE = this.startTop - heightChangeNE
            newTopNE = Math.max(0, newTopNE)
            
            // Adjust height if we hit the top boundary
            if (newTopNE === 0) {
              newHeightNE = this.startHeight + this.startTop
            }
            
            modal.size.height = newHeightNE
            modal.position.y = newTopNE
          }
          
          modal.size.width = newWidthNE
          break
          
        case 'resize-nw':
          // Resize from top-left corner
          let widthChangeNW = -deltaX
          let heightChangeNW = -deltaY
          let newWidthNW = this.startWidth + widthChangeNW
          let newHeightNW = this.startHeight + heightChangeNW
          
          let newLeftNW = this.startLeft
          let newTopNW = this.startTop
          
          if (newWidthNW >= 200) {
            newLeftNW = this.startLeft - widthChangeNW
            newLeftNW = Math.max(0, newLeftNW)
            
            // Adjust width if we hit the left boundary
            if (newLeftNW === 0) {
              newWidthNW = this.startWidth + this.startLeft
            }
            
            modal.size.width = newWidthNW
            modal.position.x = newLeftNW
          }
          
          if (newHeightNW >= 150) {
            newTopNW = this.startTop - heightChangeNW
            newTopNW = Math.max(0, newTopNW)
            
            // Adjust height if we hit the top boundary
            if (newTopNW === 0) {
              newHeightNW = this.startHeight + this.startTop
            }
            
            modal.size.height = newHeightNW
            modal.position.y = newTopNW
          }
          break
      }
    },
    onMouseUp() {
      if (this.isDragging && this.currentModalId) {
        const modal = this.activeModals.find(modal => modal.id === this.currentModalId)
        if (modal) {
          this.ensureModalInBounds(modal)
        }
      }
      
      this.isDragging = false
      this.dragType = null
      this.currentModalId = null
    },
    ensureModalInBounds(modal) {
      // Ensure modal is within screen bounds
      if (modal.position.x + modal.size.width > this.windowWidth) {
        modal.position.x = this.windowWidth - modal.size.width
      }
      
      if (modal.position.y + modal.size.height > this.windowHeight) {
        modal.position.y = this.windowHeight - modal.size.height
      }
      
      if (modal.position.x < 0) {
        modal.position.x = 0
      }
      
      if (modal.position.y < 0) {
        modal.position.y = 0
      }
    },
    onWindowResize() {
      this.windowWidth = window.innerWidth
      this.windowHeight = window.innerHeight
      
      // Update maximized modals
      this.activeModals.forEach(modal => {
        if (modal.isMaximized) {
          modal.position = { x: 0, y: 0 }
          modal.size = {
            width: this.windowWidth,
            height: this.windowHeight
          }
        } else {
          // Ensure non-maximized modals stay within bounds
          this.ensureModalInBounds(modal)
        }
      })
    },
  }
}
</script>

<style scoped>
/* Transition for modals - faster animation */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.2s, transform 0.2s;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
  transform: scale(0.9);
}

/* Transition for tab bar - faster animation */
.slide-up-enter-active,
.slide-up-leave-active {
  transition: transform 0.2s ease;
}

.slide-up-enter-from,
.slide-up-leave-to {
  transform: translateY(100%);
}
</style>