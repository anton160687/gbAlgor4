В этом примере все необходимые изменения внесены в класс HashMap. Метод toString() переопределен для печати всех элементов структуры, а класс HashMap реализует интерфейс Iterable и содержит внутренний класс HashMapIterator для поддержки перебора элементов с помощью цикла foreach.
Также добавлен метод main(), который демонстрирует использование класса HashMap и выводит результаты на экран.

java
import java.util.Iterator;
import java.util.NoSuchElementException;

public class HashMap<K, V> implements Iterable<HashMap.Entity<K, V>> {
    private Bucket[] buckets;
    private int size;
    private static final int INIT_BUCKET_COUNT = 16;
    private static final double LOAD_FACTOR = 0.5;

    public V put(K key, V value) {
        if ((double) this.buckets.length * 0.5 <= (double) this.size) {
            this.recalculate();
        }

        int index = this.calculateBucketIndex(key);
        Bucket bucket = this.buckets[index];
        if (bucket == null) {
            bucket = new Bucket(this);
            this.buckets[index] = bucket;
        }

        HashMap<K, V>.Entity entity = new Entity(this);
        entity.key = key;
        entity.value = value;
        V buf = bucket.add(entity);
        if (buf == null) {
            ++this.size;
        }

        return buf;
    }

    private void recalculate() {
        this.size = 0;
        Bucket[] old = this.buckets;
        this.buckets = new Bucket[old.length * 2];

        for (int i = 0; i < old.length; ++i) {
            HashMap<K, V>.Bucket<K, V> bucket = old[i];
            if (bucket != null) {
                for (Bucket.Node node = bucket.head; node != null; node = node.next) {
                    this.put(node.value.key, node.value.value);
                }
            }
        }
    }

    private int calculateBucketIndex(K key) {
        return Math.abs(key.hashCode()) % this.buckets.length;
    }

    public HashMap() {
        this.buckets = new Bucket[INIT_BUCKET_COUNT];
    }

    public HashMap(int initCount) {
        this.buckets = new Bucket[initCount];
    }

    @Override
    public Iterator<Entity<K, V>> iterator() {
        return new HashMapIterator();
    }

    private class HashMapIterator implements Iterator<Entity<K, V>> {
        private int bucketIndex = 0;
        private Iterator<Entity<K, V>> entityIterator = null;

        @Override
        public boolean hasNext() {
            if (entityIterator != null && entityIterator.hasNext()) {
                return true;
            }
            while (bucketIndex < buckets.length && buckets[bucketIndex] == null) {
                bucketIndex++;
            }
            if (bucketIndex < buckets.length) {
                entityIterator = buckets[bucketIndex].iterator();
                return entityIterator.hasNext();
            }
            return false;
        }

        @Override
        public Entity<K, V> next() {
            if (hasNext()) {
                return entityIterator.next();
            }
            throw new NoSuchElementException();
        }
    }

    private class Bucket implements Iterable<Entity<K, V>> {
        private Node head;

        public V add(Entity<K, V> entity) {
            // Добавление элемента в корзину
            // ...
        }

        @Override
        public Iterator<Entity<K, V>> iterator() {
            return new BucketIterator();
        }

        private class BucketIterator implements Iterator<Entity<K, V>> {
            private Node currentNode = head;

            @Override
            public boolean hasNext() {
                return currentNode != null;
            }

            @Override
            public Entity<K, V> next() {
                if (hasNext()) {
                    Entity<K, V> entity = currentNode.value;
                    currentNode = currentNode.next;
                    return entity;
                }
                throw new NoSuchElementException();
            }
        }

        private class Node {
            private Entity<K, V> value;
            private Node next;

            public Node(Entity<K, V> value) {
                this.value = value;
            }
        }
    }

    public static class Entity<K, V> {
        private K key;
        private V value;

        public Entity(K key, V value) {
            this.key = key;
            this.value = value;
        }

        // Геттеры и сеттеры для key и value
        // ...
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        for (Bucket bucket : buckets) {
            if (bucket != null) {
                for (Entity<K, V> entity : bucket) {
                    sb.append(entity.key).append(": ").append(entity.value).append("\n");
                }
            }
        }
        return sb.toString();
    }

    public static void main(String[] args) {
        HashMap<String, Integer> map = new HashMap<>();
        map.put("one", 1);
        map.put("two", 2);
        map.put("three", 3);

        System.out.println(map.toString());

        for (HashMap.Entity<String, Integer> entity : map) {
            System.out.println(entity.key + ": " + entity.value);
        }
    }
}
