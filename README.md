namespace Rpg
{
    class Character
    {
        public string Name;
        public string Camp;
        public int X;
        public int Y;
        public float Damage;
        public float Health;
        public int Lives = 3;
        public int Level = 1;
        public Character(string name, string camp, int x, int y, float damage, float health)
        {
            Name = name;
            Camp = camp;
            X = x;
            Y = y;
            Damage = damage;
            Health = health;
        }
        public virtual void PrintInfo()
        {
            Console.WriteLine($"{Name} ({Camp}) [{X},{Y}] - Здоровье: {Health}, Урон: {Damage}, Жизни: {Lives}, Уровень: {Level}");
        }
        public virtual void MoveHorizontal(int dx)
        {
            X += dx;
            if (X < 0) X = 0;
            if (X > 9) X = 9;
        }
        public virtual void MoveVertical(int dy)
        {
            Y += dy;
            if (Y < 0) Y = 0;
            if (Y > 9) Y = 9;
        }
        public virtual void TakeDamage(float totalDamage)
        {
            Health -= totalDamage;
            if (Health < 0) Health = 0;
            Console.WriteLine($"{Name} получил {totalDamage} урона. Осталось здоровья: {Health}");
            if (Health <= 0)
                OnDeath();
        }
        public virtual void OnDeath()
        {
            Lives--;
            if (Lives > 0)
            {
                Console.WriteLine($"{Name} теряет жизнь. Осталось жизней: {Lives}");
                Health = 100f;
            }
            else
            {
                Console.WriteLine($"{Name} погиб окончательно.");
            }
        }
        public bool IsAlive()
        {
            return Lives > 0;
        }
        public virtual void Defend()
        {
            Console.WriteLine($"{Name} защищается.");
        }
        public virtual void UseAbility()
        {
            Console.WriteLine($"{Name} использует способность.");
        }
    }
    class Program
    {
        static List<Character> characters = new List<Character>();
        static Random rand = new Random();
        static void Main(string[] args)
        {
            InitializeCharacters();
            while (true)
            {
                DrawField();
                var aliveCharacters = characters.FindAll(c => c.IsAlive());
                foreach (var character in aliveCharacters)
                {
                    if (!character.IsAlive()) continue;
                    int action = rand.Next(4);
                    switch (action)
                    {
                        case 0:
                            MoveRandom(character);
                            break;
                        case 1:
                            var target = ChooseTarget(character);
                            if (target != null)
                            {
                                Attack(character, target);
                            }
                            break;
                        case 2:
                            character.Defend();
                            break;
                        case 3:
                            character.UseAbility();
                            break;
                    }
                }
                characters.RemoveAll(c => !c.IsAlive());
                string winner = CheckForVictory();
                if (winner != null)
                {
                    DeclareVictory(winner);
                    break;
                }
                Console.WriteLine("\nНажмите Enter для продолжения...");
                Console.ReadLine();
            }
        }
        static void MoveRandom(Character c)
        {
            int[] directions = { -1, 0, 1 };
            int dx = directions[rand.Next(directions.Length)];
            int dy = directions[rand.Next(directions.Length)];
            int oldX = c.X;
            int oldY = c.Y;
            c.MoveHorizontal(dx);
            c.MoveVertical(dy);
            Console.WriteLine($"{c.Name} переместился из [{oldX},{oldY}] в [{c.X},{c.Y}]");
        }
        static string CheckForVictory()
        {
            bool hasRed = characters.Exists(c => c.IsAlive() && c.Camp == "Красные");
            bool hasBlue = characters.Exists(c => c.IsAlive() && c.Camp == "Синие");
            if (!hasRed) return "Синие";
            if (!hasBlue) return "Красные";
            return null;
        }
        static void InitializeCharacters()
        {
            string[] redNames = { "Арес", "Марс", "Тир", "Вулкан" };
            string[] blueNames = { "Посейдон", "Аполлон", "Гермес", "Аид" };
            for (int i = 0; i < 4; i++)
            {
                Character c = new Character(
                    redNames[i],
                    "Красные",
                    i % 2,
                    i / 2,
                    50f,
                    100f
                );
                characters.Add(c);
            }
            for (int i = 0; i < 4; i++)
            {
                Character c = new Character(
                    blueNames[i],
                    "Синие",
                    9 - (i % 2),
                    9 - (i / 2),
                    50f,
                    100f
                );
                characters.Add(c);
            }
            Console.WriteLine("Персонажи успешно размещены!");
        }
        static void DrawField()
        {
            char[,] field = new char[10, 10];
            for (int x = 0; x < 10; x++)
                for (int y = 0; y < 10; y++)
                    field[x, y] = '.';
            foreach (var c in characters)
                if (c.IsAlive())
                    field[c.X, c.Y] = c.Camp[0];
            Console.Clear();
            Console.WriteLine("Поле боя:");
            for (int x = 0; x < 10; x++)
            {
                for (int y = 0; y < 10; y++)
                    Console.Write(field[x, y] + " ");
                Console.WriteLine();
            }
            Console.WriteLine();
        }
        static Character ChooseTarget(Character attacker)
        {
            var targets = characters.FindAll(c =>
                c.IsAlive() &&
                c.Camp != attacker.Camp &&
                AreAdjacent(attacker, c));
            if (targets.Count == 0)
            {
                Console.WriteLine($"{attacker.Name} не нашёл противников рядом.");
                return null;
            }
            return targets[rand.Next(targets.Count)];
        }
        static void Attack(Character attacker, Character target)
        {
            Console.WriteLine($"\n{attacker.Name} атакует {target.Name}");
            float attackPower = GetCombinedDamage(attacker);
            target.TakeDamage(attackPower);
        }
        static float GetCombinedDamage(Character source)
        {
            float totalDamage = source.Damage;
            foreach (var c in characters)
            {
                if (c.IsAlive() && c.Camp == source.Camp && c != source && AreAdjacent(source, c))
                {
                    Console.WriteLine($"{source.Name} и {c.Name} рядом — их урон складывается!");
                    totalDamage += c.Damage;
                }
            }
            return totalDamage;
        }
        static bool AreAdjacent(Character a, Character b)
        {
            return Math.Abs(a.X - b.X) <= 1 && Math.Abs(a.Y - b.Y) <= 1;
        }
        static void DeclareVictory(string winner)
        {
            Console.WriteLine($"\nПобедил лагерь: {winner}!");
            Console.WriteLine("Оставшиеся персонажи:");
            foreach (var c in characters.FindAll(x => x.IsAlive()))
                c.PrintInfo();
            Console.WriteLine("\nИгра окончена.");
        }
    }
}
