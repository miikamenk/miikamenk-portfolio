<script setup lang="ts">
import { ref } from 'vue'

// Import all project JSON files
const projectFiles = import.meta.glob('../projects/*.json', { eager: true })
const imageFiles = import.meta.glob('../projects/images/*', { eager: true, import: 'default' })
// fallback svg image
const fallbackSvg = `data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 640 640'><path fill='%2374C0FC' d='M160 96C124.7 96 96 124.7 96 160L96 480C96 515.3 124.7 544 160 544L480 544C515.3 544 544 515.3 544 480L544 160C544 124.7 515.3 96 480 96L160 96zM224 176C250.5 176 272 197.5 272 224C272 250.5 250.5 272 224 272C197.5 272 176 250.5 176 224C176 197.5 197.5 176 224 176zM368 288C376.4 288 384.1 292.4 388.5 299.5L476.5 443.5C481 450.9 481.2 460.2 477 467.8C472.8 475.4 464.7 480 456 480L184 480C175.1 480 166.8 475 162.7 467.1C158.6 459.2 159.2 449.6 164.3 442.3L220.3 362.3C224.8 355.9 232.1 352.1 240 352.1C247.9 352.1 255.2 355.9 259.7 362.3L286.1 400.1L347.5 299.6C351.9 292.5 359.6 288.1 368 288.1z'/></svg>`

type Project = {
  title: string
  thumbnail: string
  text: string
  uploadDate: string
}

const projects = ref<Project[]>(
  Object.values(projectFiles).map((data: any) => {
    const project = data.default

    // If the thumbnail path matches an imported image, replace it with the actual URL
    for (const [path, img] of Object.entries(imageFiles)) {
      if (path.endsWith(project.thumbnail.split('/').pop()!)) {
        project.thumbnail = img as string
      }
    }

    return project
  }),
)

const selectedProject = ref<Project | null>(null)

function openProject(project: Project) {
  selectedProject.value = project
}

function closeProject() {
  selectedProject.value = null
}
</script>

<template>
  <section class="projects">
    <h2 class="section-title">Projects</h2>
    <div class="grid">
      <div
        v-for="(project, i) in projects"
        :key="i"
        class="project-card"
        @click="openProject(project)"
      >
        <img
          :src="project.thumbnail"
          :alt="project.title"
          @error="$event.target.src = fallbackSvg"
        />
        <h3>{{ project.title }}</h3>
      </div>
    </div>

    <div v-if="selectedProject" class="modal">
      <div class="modal-content">
        <button class="close-btn" @click="closeProject">✕</button>
        <h2>{{ selectedProject.title }}</h2>
        <p class="date">Uploaded: {{ selectedProject.uploadDate }}</p>
        <img
          class="modal-img"
          :src="selectedProject.thumbnail"
          @error="$event.target.style.display = 'none'"
        />
        <p class="text" v-html="selectedProject.text"></p>
      </div>
    </div>
  </section>
</template>

<style scoped>
.projects {
  width: 100%;
  max-width: 100%;
  margin: 0 auto;
  padding: 2rem;
  box-sizing: border-box;
}

.section-title {
  font-size: 2rem;
  font-weight: 700;
  margin-bottom: 1.5rem;
  text-align: center;
}

.grid {
  display: grid;
  gap: 1.5rem;
  width: 100%;
  box-sizing: border-box;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
}

@media (max-width: 768px) {
  .grid {
    grid-template-columns: 1fr;
  }
}

.project-card {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: var(--color-background-soft);
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: transform 0.2s ease;
}

.project-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
}

.project-card img {
  width: 100%;
  height: 160px;
  object-fit: cover;
}

.project-card h3 {
  padding: 0.75rem;
  text-align: center;
  font-size: 1.2rem;
}

/* Modal */
.modal {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.65);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000;
  animation: fadeIn 0.3s ease;
}

.modal-content {
  background: var(--color-background);
  padding: 2rem;
  border-radius: 12px;
  max-width: 700px;
  width: 90%;
  position: relative;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
  animation: slideUp 0.3s ease;
}

.close-btn {
  position: absolute;
  top: 0.75rem;
  right: 0.75rem;
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: var(--color-text);
}

.modal-img {
  width: 100%;
  height: auto;
  margin: 1rem 0;
  border-radius: 8px;
}

.date {
  font-size: 0.9rem;
  color: var(--color-text-secondary, #666);
  margin-bottom: 1rem;
}

.text {
  font-size: 1rem;
  line-height: 1.5;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes slideUp {
  from {
    transform: translateY(20px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

@media (min-width: 769px) {
  .grid {
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  }
}
</style>
