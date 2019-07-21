<template>
  <div class="element">
    <el-form v-model="task">
      <el-form-item>
        <el-input type="text" v-model="task.name">
          <el-button slot="append" native-type="submit" @click="addTask">ADD</el-button>
        </el-input>
      </el-form-item>
    </el-form>
    <el-table :data="tasks" height="100%" :row-class-name="dispatchRowClass">
      <el-table-column type="selection" width="50" />
      <el-table-column label="Project" width="150">
        <template slot-scope="scope">
          <el-select
            size="mini"
            @change="onProjectChanged(scope.row, $event)"
            :value="scope.row.project.name"
            filterable
            allow-create
            default-first-option
            value-key="id"
            placeholder=""
          >
            <el-option v-for="p in projects" :key="p.id" :label="p.name" :value="p.name" />
          </el-select>
        </template>
      </el-table-column>
      <el-table-column label="Task" prop="name" />
      <el-table-column label="Tags" width="120">
        <template slot-scope="scope">
          <el-tag v-for="t in scope.row.tags" :key="t.id" size="mini" closable @close="removeTag(scope.row, t)">
            {{t.name}}
          </el-tag>
          <el-select
            size="mini"
            @change="onTagChanged(scope.row, $event)"
            :value="tag.name"
            filterable
            allow-create
            default-first-option
            value-key="id"
            placeholder=""
          >
            <el-option v-for="t in tags" :key="t.id" :label="t.name" :value="t.name" />
          </el-select>
        </template>
      </el-table-column>
      <el-table-column label="Time" width="100">
        <template slot-scope="scope">
          {{calculateTime(scope.row)}}
        </template>
      </el-table-column>
      <el-table-column width="100">
        <template slot-scope="scope">
          <el-button :class="{'el-icon-video-play' : !scope.row.running, 'el-icon-video-pause': scope.row.running}" size="mini" @click="toggleRunning(scope.row)" />
        </template>
      </el-table-column>
    </el-table>
    <el-button @click="dump">DUMP</el-button>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { Button, Dialog } from "element-ui";
import { ulid } from 'ulid';
import { differenceInSeconds, differenceInHours, getSeconds, getMinutes } from 'date-fns'

interface Project {
  id: string
  name: string
}

interface Tag {
  id: string
  name: string
}

interface Task {
  id: string
  project: Project
  name: string
  tags: Tag[]
  running: boolean
  time: number
}

const emptyProject = (): Project => ({ id: '', name: '' })
const emptyTag = (): Tag => ({ id: '', name: '' })
const emptyTask = (): Task => ({ id:'', project: emptyProject(), name: '', tags: [], running: false, time: 0 })

const pad = (n: number, d: number) => {
  const s = String(n)
  return (s.length < d) ? (Array.from({length: d}).map(() => '0').join('') + s).slice(-d) : s
}

@Component({})
export default class Element extends Vue {
  task: Task = emptyTask()
  tasks: Task[] = []
  projects: Project[] = []
  tags: Tag[] = []
  creatingTag: boolean = false
  tag: Tag = emptyTag()
  start: Date = new Date()
  now: Date = new Date()
  intervalId: any = null

  addTask() {
    this.tasks.push(Object.assign({}, this.task, { id: ulid() }))
    this.task = emptyTask()
  }

  dispatchRowClass({ row }: { row: Task }) {
    if (row.running) {
      return 'task-running'
    }

    return ''
  }
  
  onProjectChanged(task: Task, value: string) {
    const found = this.projects.filter((p) => p.name === value)
    if (found.length) {
      task.project = found[0]
      return
    }

    const p = { id: ulid(), name: value }
    this.projects.push(p)
    task.project = p
  }

  onTagChanged(task: Task, value: string) {
    const found = this.tags.filter((t) => t.name === value)
    if (found.length) {
      task.tags = [
        ...task.tags.filter(t => t.id !== found[0].id),
        found[0]
      ]
      return
    }

    const t = { id: ulid(), name: value }
    this.tags.push(t)
    task.tags.push(t)
  }

  resume(task: Task, start: Date, stop: Date) {
    task.time += differenceInSeconds(stop, start)
  }

  calculateTime(task: Task) {
    const t = task.running ? task.time + differenceInSeconds(this.now, this.start) : task.time
    const td = new Date(t * 1000)
    return `${pad(differenceInHours(td, new Date(0)), 2)}:${pad(getMinutes(td), 2)}:${pad(getSeconds(td), 2)}`
  }

  toggleRunning(task: Task) {
    task.running = !task.running

    clearInterval(this.intervalId)

    if (task.running) {
      this.tasks.forEach(t => {
        if (t.id !== task.id && t.running) {
          this.resume(t, this.start, this.now)
          t.running = false
        }
      })
      
      this.start = this.now = new Date()
      this.intervalId = setInterval(() => this.now = new Date(), 1000)
    } else {
      this.resume(task, this.start, this.now)
    }
  }

  removeTag(task: Task, tag: Tag) {
    task.tags = task.tags.filter(t => t.id !== tag.id)
  }

  startCreateTag() {
    this.creatingTag = true
  }

  endCreateTag(task: Task) {
    const tagName = this.tag.name.trim()
    this.creatingTag = false
    this.tag = emptyTag()

    if (!tagName) {
      return
    }

    const found = this.tags.filter(t => t.name === tagName)
    if (found.length) {
      if (!task.tags.some(t => t.id === found[0].id)) {
        task.tags.push(found[0])
      }

      return
    }

    const t = { id: ulid(), name: tagName }
    this.tags.push(t)
    task.tags.push(t)
  }

  dump() {
    console.log(
      JSON.stringify(
        {
          tasks: this.tasks,
          projects: this.projects,
          tags: this.tags
        },
        null,
        2
      )
    )
  }
}
</script>

<style>

.element {
  display: flex;
  flex-direction: column;
}

.el-table .task-running {
  background: #f0f9eb;
}

</style>