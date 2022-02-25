# Features
- Go-routines to execute tasks independently, maybe in parallel.
- Channels for communication, synchronization b/w go-routines.

# Bla..bla..bla

func main() {
    tasks := getTasks()

    for _, task := range tasks {
        process(task)
    }
}
--------------------------------------

func main() {
    ch := make(chan Task, 3)

    for i:=0; i<3; i++ {
        go worker(ch)
    }

    taskList := getTasks()

    for _, task := range taskList {
        ch <- task
    }
}

func worker(ch <-chan Task) {
    for {
        task := <-ch
        process(task)
    }
}

# Channel Properties
- go-routine safe
- buffer can store values and pass values among go-routines
- FIFO semantics
- Can cause go-routines to block and un-block