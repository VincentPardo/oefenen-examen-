# oefenen-examen- zelf gemaakt programma gebasseerd op oefen_examen
import csv

class ProjectTask:
    def __init__(self, task_id: str, project_name: str, task_description: str, estimated_hours: int, priority: int, assigned_team: str):
        self.task_id = task_id
        self.project_name = project_name
        self.task_description = task_description
        self.estimated_hours = estimated_hours
        self.priority = priority  # CORRECTIE: was 'prioriy'
        self.assigned_team = assigned_team
        self.next = None

class ProjectQueue:
    def __init__(self):
        self.head = None
    
    def add_task(self, task_id, project_name, task_description, estimated_hours, priority, assigned_team):
        new_task = ProjectTask(task_id, project_name, task_description, estimated_hours, priority, assigned_team)
        if self.head is None:
            self.head = new_task
            return  # CORRECTIE: return toevoegen om infinite loop te voorkomen
        
        current = self.head
        while current.next is not None:  # CORRECTIE: None i.p.v. None
            current = current.next
        current.next = new_task
    
    def remove_task(self, task_id):
        if self.head is None:
            return
        
        # CORRECTIE: special case voor head verwijderen
        if self.head.task_id == task_id:
            self.head = self.head.next
            return
        
        current = self.head
        previous = None
        while current:
            if current.task_id == task_id:
                if previous:
                    previous.next = current.next
                return
            previous = current
            current = current.next
    
    def display_tasks(self):
        current = self.head
        while current:
            print(f"{current.task_id} : {current.project_name} : {current.task_description} : {current.estimated_hours} : {current.priority} : {current.assigned_team}")
            current = current.next
    
    def find_tasks_by_project(self, project_name):  # CORRECTIE: functienaam consistent maken
        current = self.head
        taken_bij_project = []
        while current:
            if current.project_name == project_name:
                taken_bij_project.append(current)
            current = current.next
        return taken_bij_project  # CORRECTIE: return de lijst, niet een string
    
    def calculate_total_hours(self):
        total_hours = 0
        current = self.head
        while current:
            total_hours += current.estimated_hours
            current = current.next
        return total_hours
    
    def read_tasks_from_csv(self, file_path):
        try:
            with open(file_path, 'r', encoding='utf-8') as f:
                reader = csv.reader(f, delimiter=',')
                next(reader)  # CORRECTIE: header overslaan indien aanwezig
                for row in reader:
                    if len(row) >= 6:  # CORRECTIE: controle op aantal kolommen
                        task_id = row[0]
                        project_name = row[1]
                        task_description = row[2]
                        estimated_hours = int(row[3])
                        priority = int(row[4])
                        assigned_team = row[5]
                        self.add_task(task_id, project_name, task_description, estimated_hours, priority, assigned_team)
        except FileNotFoundError:
            print(f"Bestand {file_path} niet gevonden")
        except Exception as e:
            print(f"Fout bij lezen CSV: {e}")
    
    def sorted_insert_by_priority(self, head, node):
        """Hulpmethode voor gesorteerd invoegen op prioriteit"""
        # Maak een kopie van de node om de originele lijst niet te beschadigen
        new_node = ProjectTask(node.task_id, node.project_name, node.task_description, 
                              node.estimated_hours, node.priority, node.assigned_team)
        
        if head is None or new_node.priority < head.priority:
            new_node.next = head
            return new_node
        
        current = head
        while current.next is not None and current.next.priority <= new_node.priority:
            current = current.next
        
        new_node.next = current.next
        current.next = new_node
        return head
    
    def reorder_tasks_by_priority(self):  # CORRECTIE: functienaam consistent
        if self.head is None:
            return
        
        new_head = None
        current = self.head
        
        while current:
            next_temp = current.next  # Bewaar volgende pointer
            current.next = None  # Isoleer current node
            new_head = self.sorted_insert_by_priority(new_head, current)
            current = next_temp
        
        self.head = new_head
    
    def sorted_insert_by_project_priority(self, head, node):
        """Hulpmethode voor gesorteerd invoegen op project en prioriteit"""
        new_node = ProjectTask(node.task_id, node.project_name, node.task_description, 
                              node.estimated_hours, node.priority, node.assigned_team)
        
        if head is None:
            return new_node
        
        # Vergelijk eerst projectnaam, dan prioriteit
        if (new_node.project_name < head.project_name or 
            (new_node.project_name == head.project_name and new_node.priority < head.priority)):
            new_node.next = head
            return new_node
        
        current = head
        while current.next is not None:
            # Bepaal of nieuwe node voor current.next moet komen
            if (new_node.project_name < current.next.project_name or 
                (new_node.project_name == current.next.project_name and new_node.priority < current.next.priority)):
                break
            current = current.next
        
        new_node.next = current.next
        current.next = new_node
        return head
    
    def reorder_tasks_by_project_priority(self):  # CORRECTIE: ontbrekende dubbele punt
        if self.head is None:
            return
        
        new_head = None
        current = self.head
        
        while current:
            next_temp = current.next
            current.next = None
            new_head = self.sorted_insert_by_project_priority(new_head, current)
            current = next_temp
        
        self.head = new_head
    
    def optimize_team_workload(self):
        """Optimaliseer teamwerkbelasting - groepeer per team en sorteer op prioriteit"""
        if self.head is None:
            return self
        
        # Groepeer taken per team
        teams = {}
        current = self.head
        while current:
            team = current.assigned_team
            if team not in teams:
                teams[team] = []
            teams[team].append(current)
            current = current.next
        
        # Sort each team's tasks by priority
        for team in teams:
            teams[team].sort(key=lambda task: task.priority)
        
        # Bouw nieuwe gelinkte lijst
        new_head = None
        current_tail = None
        
        for team in sorted(teams.keys()):  # Teams in alfabetische volgorde
            for task in teams[team]:
                new_node = ProjectTask(task.task_id, task.project_name, task.task_description,
                                     task.estimated_hours, task.priority, task.assigned_team)
                
                if new_head is None:
                    new_head = new_node
                    current_tail = new_node
                else:
                    current_tail.next = new_node
                    current_tail = new_node
        
        self.head = new_head
        return self

def main():
    project_queue = ProjectQueue()
    project_queue.read_tasks_from_csv("tasks.csv")  # CORRECTIE: echte bestandsnaam
    print("Originele taken:")
    project_queue.display_tasks()
    
    print(f"\nTotaal geschatte uren: {project_queue.calculate_total_hours()}")
    
    # Test find_tasks_by_project
    project_tasks = project_queue.find_tasks_by_project("Project Alpha")
    print(f"\nTaken voor Project Alpha: {len(project_tasks)}")
    
    # Test reorder by priority
    project_queue.reorder_tasks_by_priority()
    print("\nGesorteerd op prioriteit:")
    project_queue.display_tasks()
    
    # Test reorder by project and priority
    project_queue.reorder_tasks_by_project_priority()
    print("\nGesorteerd op project en prioriteit:")
    project_queue.display_tasks()
    
    # Test team workload optimization
    project_queue.optimize_team_workload()
    print("\nGeoptimaliseerde teamwerkbelasting:")
    project_queue.display_tasks()

if __name__ == "__main__":
    main()
