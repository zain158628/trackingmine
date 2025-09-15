# trackingmine
its is an family tracking app which can be use to track your loveone if they are not on house 
// Tracking App — Single-file React starter
// Features: add tracking items (packages/tasks), update status, timeline, simple search, localStorage persistence
// Tailwind CSS classes used for styling (no imports required in canvas preview). Default export component.

import React, { useState, useEffect } from "react";

// Helper: generate id
const uid = () => Math.random().toString(36).slice(2, 9);

const defaultItems = [
  {
    id: uid(),
    title: "Parcel — Order #12345",
    description: "Expected delivery: 2025-09-20",
    status: "In Transit",
    history: [
      { ts: Date.now() - 1000 * 60 * 60 * 24 * 5, note: "Shipped" },
      { ts: Date.now() - 1000 * 60 * 60 * 24 * 2, note: "Arrived at sorting center" },
    ],
  },
  {
    id: uid(),
    title: "Project milestone",
    description: "Phase 2 — QA",
    status: "Pending",
    history: [{ ts: Date.now() - 1000 * 60 * 60 * 24 * 1, note: "Assigned" }],
  },
];

export default function App() {
  const [items, setItems] = useState(() => {
    try {
      const raw = localStorage.getItem("tracking_items_v1");
      return raw ? JSON.parse(raw) : defaultItems;
    } catch (e) {
      return defaultItems;
    }
  });

  const [query, setQuery] = useState("");
  const [form, setForm] = useState({ title: "", description: "" });
  const [selected, setSelected] = useState(null);

  useEffect(() => {
    localStorage.setItem("tracking_items_v1", JSON.stringify(items));
  }, [items]);

  function addItem(e) {
    e.preventDefault();
    if (!form.title.trim()) return;
    const newItem = {
      id: uid(),
      title: form.title.trim(),
      description: form.description.trim(),
      status: "Pending",
      history: [{ ts: Date.now(), note: "Created" }],
    };
    setItems([newItem, ...items]);
    setForm({ title: "", description: "" });
  }

  function updateStatus(id, newStatus, note) {
    setItems((prev) =>
      prev.map((it) =>
        it.id === id
          ? {
              ...it,
              status: newStatus,
              history: [{ ts: Date.now(), note }, ...it.history],
            }
          : it
      )
    );
  }

  function deleteItem(id) {
    if (!confirm("Delete this tracking item?")) return;
    setItems((prev) => prev.filter((i) => i.id !== id));
    if (selected === id) setSelected(null);
  }

  const filtered = items.filter(
    (it) =>
      it.title.toLowerCase().includes(query.toLowerCase()) ||
      it.description.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <div className="min-h-screen bg-gray-50 p-6">
      <div className="max-w-4xl mx-auto">
        <header className="mb-6">
          <h1 className="text-2xl font-bold">Tracking App — Starter</h1>
          <p className="text-sm text-gray-600">Add items, update status, view history. Saves to localStorage.</p>
        </header>

        <section className="grid grid-cols-1 md:grid-cols-3 gap-4">
          <form onSubmit={addItem} className="col-span-1 md:col-span-1 bg-white p-4 rounded shadow-sm">
            <h2 className="font-semibold mb-2">Add Tracking Item</h2>
            <input
              className="w-full mb-2 p-2 border rounded"
              placeholder="Title (e.g. Parcel #123)"
              value={form.title}
              onChange={(e) => setForm({ ...form, title: e.target.value })}
            />
            <textarea
              className="w-full mb-2 p-2 border rounded"
              placeholder="Short description"
              value={form.description}
              onChange={(e) => setForm({ ...form, description: e.target.value })}
            />
            <div className="flex gap-2">
              <button className="px-3 py-2 bg-blue-600 text-white rounded" type="submit">
                Add
              </button>
              <button
                type="button"
                className="px-3 py-2 bg-gray-200 rounded"
                onClick={() => setForm({ title: "", description: "" })}
              >
                Clear
              </button>
            </div>
          </form>

          <div className="col-span-2">
            <div className="bg-white p-4 rounded shadow-sm mb-4 flex items-center justify-between">
              <input
                className="flex-1 p-2 border rounded mr-4"
                placeholder="Search by title or description"
                value={query}
                onChange={(e) => setQuery(e.target.value)}
              />
              <div className="text-sm text-gray-600">{filtered.length} results</div>
            </div>

            <div className="bg-white p-4 rounded shadow-sm">
              {filtered.length === 0 && <div className="text-gray-500">No items found.</div>}

              <ul className="space-y-3">
                {filtered.map((it) => (
                  <li
                    key={it.id}
                    className={`p-3 rounded border cursor-pointer hover:bg-gray-50 flex justify-between items-center ${selected === it.id ? "ring-2 ring-blue-200" : ""}`}
                    onClick={() => setSelected(it.id)}
                  >
                    <div>
                      <div className="font-semibold">{it.title}</div>
                      <div className="text-xs text-gray-500">{it.description}</div>
                      <div className="text-xs mt-1">
                        <span className="px-2 py-1 text-xs rounded bg-yellow-100">{it.status}</span>
                      </div>
                    </div>
                    <div className="text-sm text-gray-400">{new Date(it.history[0]?.ts || Date.now()).toLocaleDateString()}</div>
                  </li>
                ))}
              </ul>
            </div>

            {selected && (
              <div className="mt-4 bg-white p-4 rounded shadow-sm">
                {(() => {
                  const item = items.find((x) => x.id === selected);
                  if (!item) return null;
                  return (
                    <div>
                      <div className="flex justify-between items-start">
                        <div>
                          <h3 className="text-lg font-semibold">{item.title}</h3>
                          <p className="text-sm text-gray-600">{item.description}</p>
                          <div className="mt-2">Status: <strong>{item.status}</strong></div>
                        </div>
                        <div className="text-right">
                          <button
                            className="px-2 py-1 text-sm bg-red-100 rounded mr-2"
                            onClick={() => deleteItem(item.id)}
                          >
                            Delete
                          </button>
                        </div>
                      </div>
